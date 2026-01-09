# PlatformTransactionManager
```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition)
            throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

### 핵심 개념

* **TransactionDefinition**

    * 전파(propagation), 격리(isolation), timeout, readOnly
* **TransactionStatus**

    * 현재 트랜잭션 상태 (신규 여부, 롤백 전용 여부 등)

# AbstractPlatformTransactionManager

```java
public abstract class AbstractPlatformTransactionManager
        implements PlatformTransactionManager
```

## 주요 필드

```java
private boolean nestedTransactionAllowed = false;
private boolean globalRollbackOnParticipationFailure = true;
private boolean rollbackOnCommitFailure = false;
```

> 전파, 중첩, 예외 시 정책 제어

# getTransaction() 내부 흐름

```java
public final TransactionStatus getTransaction(TransactionDefinition definition)
```

### 전체 흐름 요약

```
1. 기존 트랜잭션 존재 여부 확인
2. 전파 옵션에 따라 분기
3. 필요 시 신규 트랜잭션 생성
4. ThreadLocal에 리소스 바인딩
```

## 기존 트랜잭션 확인

```java
Object transaction = doGetTransaction();
boolean existingTransaction = isExistingTransaction(transaction);
```

### doGetTransaction()

* 구현체(JpaTransactionManager 등)가 제공
* 보통 **ConnectionHolder / EntityManagerHolder** 반환

### isExistingTransaction()

* 리소스가 이미 활성 상태인지 판단

> 여기서 **ThreadLocal 기반 구조**가 시작됨

## 전파 속성에 따른 분기

### REQUIRED (기본값)

```java
if (existingTransaction) {
    return prepareTransactionStatus(...);
}
```

* 기존 트랜잭션 참여
* **새로운 DB 트랜잭션 생성 안 함**


### REQUIRES_NEW

```java
suspend(transaction);
startTransaction();
```

* 기존 트랜잭션 **suspend**
* 새 트랜잭션 생성
* 커밋/롤백 독립

> 실무에서 데드락·예상치 못한 롤백의 주범


### NESTED

```java
if (!nestedTransactionAllowed) {
    throw new NestedTransactionNotSupportedException();
}
```

* JDBC Savepoint 사용
* **JPA는 사실상 거의 미지원**


### NOT_SUPPORTED

```java
suspend(transaction);
```

* 트랜잭션 없이 실행


## 신규 트랜잭션 생성

```java
doBegin(transaction, definition);
```

### 여기서 일어나는 일

* DB Connection 획득
* autoCommit = false
* isolation level 설정
* timeout 설정


## ThreadLocal 바인딩

```java
TransactionSynchronizationManager.bindResource(dataSource, connectionHolder);
```

### 핵심 클래스

```java
TransactionSynchronizationManager
```

* **ThreadLocal Map** 사용
* key: DataSource / EntityManagerFactory
* value: ConnectionHolder / EntityManagerHolder

>이 덕분에 **같은 스레드에서는 어디서든 같은 커넥션을 사용**


# commit() 내부 동작

```java
public final void commit(TransactionStatus status)
```

## 롤백 전용 체크

```java
if (status.isRollbackOnly()) {
    processRollback(status);
    return;
}
```

### rollback-only가 설정되는 경우

* 내부 트랜잭션 예외 발생
* REQUIRED 참여 중 예외

> **UnexpectedRollbackException의 원인**


## 실제 커밋 분기

```java
if (status.isNewTransaction()) {
    doCommit(status);
}
```

* 신규 트랜잭션만 실제 DB commit
* 참여 트랜잭션은 아무 것도 안 함



## Synchronization 콜백 실행

```java
triggerBeforeCommit();
triggerAfterCommit();
triggerAfterCompletion();
```

### 여기서 실행되는 것들

* `@TransactionalEventListener`
* MyBatis flush
* JPA flush

> 커밋 타이밍 이슈 대부분 여기서 발생

## 리소스 정리

```java
cleanupAfterCompletion(status);
```

* Connection close
* ThreadLocal unbind
* suspend 된 트랜잭션 resume


# rollback() 내부 동작

```java
public final void rollback(TransactionStatus status)
```

## 신규 트랜잭션

```java
doRollback(status);
```

> DB rollback 실행

## 참여 트랜잭션

```java
doSetRollbackOnly(status);
```

> **실제 롤백은 상위 트랜잭션에서**

### 이게 중요한 이유

```java
@Transactional
public void outer() {
    inner(); // 예외 발생
}
```

* inner는 REQUIRED
* inner 예외 → rollback-only
* outer 정상 종료 → commit 시점에 **UnexpectedRollbackException**


# 실제 구현체

## DataSourceTransactionManager

```java
protected void doBegin(...) {
    Connection con = dataSource.getConnection();
    con.setAutoCommit(false);
}
```

* 순수 JDBC
* Savepoint 지원


## JpaTransactionManager

```java
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
```

* EntityManager를 ThreadLocal에 바인딩
* flush는 commit 직전

# 7. ThreadLocal 구조 요약

```
Thread
 └── TransactionSynchronizationManager
      ├── resources
      │    └── DataSource → ConnectionHolder
      ├── synchronizations
      └── transactionName
```

> **비동기 / CompletableFuture / 새로운 스레드**  
> 트랜잭션 전파 안 됨


# 실무에서 자주 터지는 포인트

### 여러 TxManager 공존

* JPA + MyBatis
* 잘못된 @Primary / @Transactional 지정

`LazyConnectionDataSourceProxy already has a holder` 오류


### 내부 호출 self-invocation

* 프록시 미적용
* 트랜잭션 아예 시작 안 됨


### REQUIRES_NEW 남용

* 커넥션 고갈
* 데드락
* 성능 저하


### readOnly = true 오해

* **DB 수준 제약 아님**
* flush 최적화 힌트 수준
