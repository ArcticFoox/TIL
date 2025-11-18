# JPA lazyLoading

엔티티 프록시(entity proxy)

@ManyToOne, @OneToOne 등 단일 연관관계에서 사용

실제 엔티티 대신 식별자(id)를 가진 프록시 객체를 반환하고, 실제 필드(이름 등)에 접근할 때 DB에서 로딩

컬렉션 프록시(collection proxy): @OneToMany, @ManyToMany 같은 컬렉션은 `PersistentBag` , `PersistentSet` 등 컬렉션 구현체로 프록시 되어 컬렉션 내용에 접근(size(), iterator())할 때 로딩

## 프록시 생성시점

`em.find()` 또는 JPQL로 `Comment`를 조회할 때, `Comment.post`가 `LAZY` 이면 Post 객체 대신 프록시 객체를 즉시 할당

프록시에는 주로 식별자(PK)가 저장되어 있음

`proxy.getId()` 처럼 식별자 접근은 추가 SELECT 없이 가능

```bash
Comment c = em.find(Comment.class, commentId);
```

## 프록시 구현 방식

### HibernateProxy(기본 방식)

런타임에 동적으로 생성된 프록시 클래스(대부분 실제 엔티티의 서브클래스 형태)를 반환

프록시 내부에 `LazyInitializer`가 있어 실제 엔티티를 불러오는 책임을 짐

### Bytecode enhancement(빌드타임/런타임 계층 변경)

엔티티 클래스에 바이트코드 변형을 적용하면 프록시가 아닌 “지연 로딩 가능한 필드 접근” 방식으로 동작(필드 수준의 지연 로딩이 가능)

## 프록시 초기화 시점

프록시를 실제 엔티티로 바꾸는(혹은 내부 구현이 엔티티를 로드하는) 트리거는 프록시의 비-식별자 멤버에 접근할 때

예시로 `c.getPost().getTitle() → getTitle()` 호출 시 Post 프록시가 초기화되고 `SELECT * FROM post WHERE id = ?` 가 호출됨

반면 `c.getPost().getId()` 는 대부분 초기화 없이 id 반환(프록시에 내장되어 있기 때문)

`comment.getReplies().size()` 등 컬렉션 내용을 확인하려 할 때도 로딩

## 세션(or EntityManager) 상태와 LazyInitializationException

프록시 초기화는 유효한 Hibernate Session / JPA EntityManager 가 열려 있을 때만 성공

트랜잭션이 끝나고(또는 영속성 컨텍스트가 닫힌 후) 프록시를 초기화하려 하면 `LazyInitializationException` 가 발생 → 컨트롤러에서 `@Transactional` 없이 뷰 렌더링 시점에 `c.getPost().getTitle()` 호출하면 예외 발생

예외 방지를 위해 필요한 연관관계는 트랜잭션 안에서 미리 초기화하거나(or fetch join 사용), DTO로 변환해서 필요한 데이터만 꺼내야한다.

## 요약

1. 엔티티 로드 → 연관은 프록시로 대체(식별자 포함)
2. 프록시의 식별자 접근은 SQL 없이 가능
3. 프록시의 비-식별자 접근(필드/메서드)은 초기화 트리거 → 유효한 Session이 있으면 DB에서 SELECT 실행하여 실체를 로드
4. 초기화 완료 후 프록시는 실제 엔티티(또는 내부 구현의 구현체)로 연결되어 더 이상 추가 SQL 발생 안함(동일 세션 내)
5. 세션이 닫힌 뒤 접근하면 `LazyInitializationException` 발생