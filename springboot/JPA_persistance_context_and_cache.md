# JPA 영속성 컨텍스트

## 영속성 컨텍스트

JPA를 사용해 엔티티(객체)를 db에 저장하기 전에 항상 영속성 컨텍스트에 먼저 저장

엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리

일반적으로 엔티티 메니저는 사용자 요청 당 1개가 생성되며, 각 엔티티 매니저마다 영속성 컨텍스트 1개가 생성

![image.png](/springboot/img/cache/image.png)

- **비영속(new/transient)** : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
    
    ```java
    // 엔티티 객체 생성. 예제를 단순화하기 위해 setter 사용
    Member member = new Member();
    member.setId(1L);
    member.setName("회원1");
    ```
    
- **영속(managed)** : 영속성 컨텍스트에 관리되는 상태
엔티티를 식별자(@Id) 값으로 구분
    
    ```java
    // em == EntityManager의 참조 변수
    em.persist(member);           // entity를 영속성 컨텍스트에 저장(영속 상태로 만듬)
    em.find(Member.class, id);    // entity를 DB에서 로드(영속 상태로 만듬)
    ```
    
- **준영속(detached)** : 영속성 컨텍스트에 저장되었다가 분리된 상태
영속성 컨텍스트에서 떼어내는 것
    
    ```java
    em.detach(member);  // 회원 엔티티를 영속성 컨텍스트에서 분리
    em.clear();         // 영속성 컨텍스트 초기화
    em.close();         // 엔티티 매니저(영속성 컨텍스트) 종료
    ```
    
- **삭제(removed)** : 삭제된 상태
엔티티를 영속성 컨텍스트와 데이터베이스(flush() 호출 시) 삭제
    
    ```java
    em.remove(member)
    ```
    

## 1차 캐시

영속성 컨텍스트는 내부에 엔티티를 저장할 수 있는 캐시를 가지고 있는데 이를 1차 캐시라고 함

엔티티를 처음 영속성 컨텍스에 저장하면(영속 상태로 만들면) 1차 캐시에 엔티티가 저장

```java
// 엔티티 객체 생성
Member member = new Member();
member.setId("member1");
member.setName("회원1");

em.persist(member);
```

키는 식별자 값(@Id)이고 값은 엔티티 인스턴스, 조회 시 DB를 거지치 않고 메모리 상에서 바로 조회

### 1차 캐시에 없는 경우

```java
Member member2 = em.find(Member.class, "member2");
```

![image.png](/springboot/img/cache/image%201.png)

1. @Id 값이 member2인 Member 엔티티가 1차 캐시에 있는지 조회
2. 1차 캐시에 없으므로 DB 조회
3. 조회한 데이터로 엔티티 생성해 1차 캐시에 저장
4. 조회한 엔티티 반환

### 동일성 보장

**동일성(Identity)**은 실제 인스턴스(인스턴스 주소)가 같다는 의미로 참조 값을 비교하는 `==` 사용

**동등성(equality)**은 실제 인스턴스가 가지고 있는 값이 같다는 의미로 `equals()` 메서드를 사용

영속성 컨텍스트에서 관리되는 엔티티 비교 시, @Id 값이 같다면 1차 캐시에 있는 동일한 엔티티 인스턴스를 반환

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
```

위 코드에서 `a == b` 를 해보면 `true` 반환

영속성 컨텍스트를 사용하지 않고 DB를 바로 거쳐 인스턴스 생성 시, 매번 새로운 인스턴스가 생성되어 동일성이 보장되지 않을 것

### 트랜잭션을 지원하는 쓰기 지연

엔티티 매니저가 관리하는 엔티티의 모든 변경은 트랜잭션 안에서 이루어 져야 함

```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();
tx.begin();

em.persist(memberA);
em.persist(memberB);

tx.commit();
```

위 코드 실행 시, `em.persist()` 를 호출한다고 해서 DB에 `INSERT` 쿼리를 날리지 않음

엔티티 매니저는 `commit()` 이 호출되기 전까지 내부 쿼리 저장소에 실행된 메서드에 해당하는 SQL들을 모아 둠

![image.png](/springboot/img/cache/image%202.png)

이것이 **트랜잭션을 지원하는 쓰기 지연(treansactional write-behind)**

### 변경감지(Dirty Checking)

JPA에서는 엔티티 수정 시 `update()` 메서드가 불 필요

단지 영속성 컨텍스트 안의 엔티티를 수정하면 DB에 자동으로 반영

JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 스냅샷으로 저장 그러다 플러시(flush())가 호출되는 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾음

### flush()

영속성 컨텍스트의 변경 내용을 DB에 반영하는 기능 → 영속성 컨텍스트를 DB와 동기화하는 작업

**동작과정**

1. 변경 감지가 동작해 영속성 컨텍스트 안에 있는 모든 엔티티를 스냅샷과 비교
2. 수정된 엔티티가 있으면 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 저장
3. 이후 `commit()` 이 호출되면 DB에 전송

**호출방법**

- 직접호출 : 엔티티 매니저의 flush() 메서드를 직접 호출 시
- 트랜잭션 커밋 시 플러시 자동 호출: 트랜잭션이 커밋되기 전에 JPA는 영속성 컨텍스트의 변경 내용을 DB에 반영하기 위해 자동 호출
- JPQL 쿼리 실행 시 플러시 자동 호출

## 2차 캐시

애플리케이션에서 공유하는 캐시를 JPA는 공유 캐시(shared cache)라 하는데 일반적으로 2차 캐시라고도 부름

애플리케이션을 종료할 때까지 유지 → 분산 캐시나 클러스터링 환경의 캐시는 애플리케이션보다 더 오래 유지될 수도 있음

1차 캐시 조회 후 없으면 2차 캐시 조회

2차 캐시는 동시성을 극대화하기 위해 캐시 한 객체를 직접 반환하지 않고 복사본을 만들어 반환 → 캐시한 객체를 그대로 반환하면 여러 곳에서 같은 객체를 동시에 수정하는 문제가 발생 할 수 있는데 이 문제를 해결하기 위해 객체에 락을 걸명 동시성이 떨어지는 문제가 있기 때문. 락에 비하면 객체를 복사하는 비용이 저렴