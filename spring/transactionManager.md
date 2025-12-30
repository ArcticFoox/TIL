# Transaction Manager 주의점
## DataSource 가 여러 개인 경우
```java
@Bean
public PlatformTransactionManager primaryTxManager() { ... }

@Bean
public PlatformTransactionManager batchTxManager() { ... }
```

```java
@Transactional(transactionManager = "primaryTxManager")
public void save() { ... }

//or

@Transactional("primaryTxManager")
```

> **서로 다른 TxManager가 같은 DataSource(LazyConnectionDataSourceProxy)를
> 동시에 트랜잭션 바인딩하려고 해서 발생하는 충돌**할 수 있음


## 오류 메시지

```
IllegalStateException:
Already value [ConnectionHolder@xxxx] for key
[org.springframework.jdbc.datasource.ConnectionHolder]
bound to thread
```

또는

```
LazyConnectionDataSourceProxy is already bound to a thread
```


> **“현재 스레드(ThreadLocal)에 이미 이 DataSource에 대한 ConnectionHolder가 있는데
> 또 하나를 바인딩하려고 한다”**


### LazyConnectionDataSourceProxy의 역할

* 트랜잭션이 시작될 때
* **즉시 DB 커넥션을 얻지 않고**
* 실제 SQL 실행 시점까지 커넥션 획득을 지연

**두 TxManager가 “같은 프록시 DataSource”를 공유**


### 시나리오

```java
@Transactional
public void service() {
    jpaRepository.save(...);
    myBatisMapper.insert(...); 
}
```

### 내부 흐름

1. `@Transactional(jpaTxManager)`
2. `JpaTransactionManager`가 트랜잭션 시작
3. `TransactionSynchronizationManager`에 등록

```text
ThreadLocal:
  DataSource(LazyProxy) → ConnectionHolder #1
```

4. 이후 MyBatis 호출
5. MyBatis는 `DataSourceTransactionManager`를 통해
6. **같은 LazyConnectionDataSourceProxy에 대해**
7. **새 ConnectionHolder를 바인딩하려고 시도**

```text
ThreadLocal:
  DataSource(LazyProxy) → ConnectionHolder #1 (이미 있음)
```

결과:

> **“이미 holder를 가지고 있다” 예외 발생**



## 근본 원인
### TxManager가 다르다

* JPA → `JpaTransactionManager`
* MyBatis → `DataSourceTransactionManager`

**각각 자기만의 트랜잭션이라고 생각함**


### DataSource는 같다

```text
JpaTransactionManager
DataSourceTransactionManager
        ↓
LazyConnectionDataSourceProxy
        ↓
Real DataSource
```

* TxManager는 다르지만
* **ThreadLocal 키는 DataSource 기준**


### LazyConnectionDataSourceProxy는 “단일 holder”만 허용

* 설계상 **한 스레드 = 한 ConnectionHolder**
* 다중 TxManager 동시 바인딩 불가


## @Primary로 해결 안 되는 이유

* 이미 **트랜잭션이 시작된 이후**
* 다른 TxManager가 개입
* Primary는 **빈 선택 단계에서만 의미**

**런타임 충돌**이기 때문에 무력

## 문제상황

### 하나의 @Transactional 안에서 JPA + MyBatis 혼용

```java
@Transactional("jpaTxManager")
public void doAll() {
    jpa.save();
    mybatis.insert(); // 오류 발생
}
```


### 해결책 트랜잭션 경계 자체를 분리

```java
@Transactional("jpaTxManager")
public void jpaWork() { ... }

@Transactional("mybatisTxManager")
public void mybatisWork() { ... }
```

