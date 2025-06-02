# Transaction

더 이상 쪼갤 수 없는 최소 작업 단위

<br>

## Spring 트랜잭션 핵심 기술 3가지

1. 트랜잭션 동기화
2. 트랜잭션 추상화
3. AOP를 이용한 트랜잭션 분리

### 트랜잭션 동기화

트랜잭션을 시작하기 위한 Connection 객체를 특별한 저장소에 보관해두고 필요할 때 꺼내 쓸 수 있도록 하는 기술

트랜잭션 동기화 저장소는 작업 쓰레드마다 Connection 객체를 독립적으로 관리하기 때문에,

멀티쓰레드 환경에서도 충돌이 발생할 여지가 없다.

### 트랜잭션 추상화

트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공

JDBC, JPA, Hibernate 등 종속적인 코드를 이용하지 않고도 일관되게 트랜잭션을 처리

![image.png](/spring/img/image3.png)

### AOP를 이용한 트랜잭션 분리

트랜잭션 코드와 비지니스 로직 코드가 복잡하게 얽혀있는 코드 분리 가능

```java
public void addUsers(List<User> userList) {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		for (User user: userList) {
			if(isEmailNotDuplicated(user.getEmail())){
				userRepository.save(user);
			}
		}

		this.transactionManager.commit(status);
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e
	}
}
```

해당 로직을 클래스 밖으로 빼내서 별도의 모듈로 만드는 AOP 고안 및 적용

```java
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {

    private final UserRepository userRepository;

    public void addUsers(List<User> userList) {
        for (User user : userList) {
            if (isEmailNotDuplicated(user.getEmail())) {
                userRepository.save(user);
            }
        }
    }
}
```

## Spring 트랜잭션 세부 설정

### 트랜잭션 전파

트랜잭션의 경계에서 이미 진행중인 트랜잭션이 있거나 없을 때 어떻게 동작할 것인가를 결정하는 방식

<br>

**트랜잭션 참여(PROPAGATION_REQUIRED)**

새로운 트랜잭션을 만들지 않고 진행중인 트랜잭션에 참여

하나로 묶여있기에 둘 중 하나에서 예외 발생 시 작업이 모두 취소됨

<br>

**독립적인 트랜잭션 생성(PROPAGATION_REQUIRES_NEW)**

경계를 빠져나오는 순간 독자적으로 커밋 또는 롤백

서로에게 어떠한 영향도 주지 않음

<br>

**트랜잭션 없이 동작(PROPAGATION_NOT_SUPPORTED)**

단순 데이터 조회 시 굳이 트랜잭션이 필요 없을 것

### 격리 수준

모든 DB 트랜잭션은 격리수준을 가져야 함

서버에서는 여러 개의 트랜잭션이 동시에 진행될 수 있는데,

모든 트랜잭션을 독립적으로 만들고 순차 진행 한다면 안전하지만 성능 감소

적절하게 격리수준을 조정해 가능한 많은 트랜잭션을 동시에 진행하면서 제어해야 함

JDBC 드라이버나 DataSource 등에서 재설정 또는 트랜잭션 단위로 격리 수준 조정

DefaultTransactionDefinition에 설정된 격리수준은 ISOLATION_DEFAULT로 DataSource에 정의된 것을 따름

특별한 작업 수행이 목적이면 독자적으로 지정해줄 필요가 있음

### 제한시간

트랜잭션을 수행하는 제한시간을 설정할 수 있음

제한시간 설정은 트랜잭션을 직접 시작하는 PROPAGATION_REQUIRED, PROPAGATION_REQUIRES_NEW 경우에 사용해야만 의미가 있음

### 읽기전용

트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있을 뿐만 아니라 데이터 액세스 기술에 따라 성능 향상

출처

[https://mangkyu.tistory.com/154](https://mangkyu.tistory.com/154)