# 영속성 컨텍스트

Persistence Context는 JPA(EntityManager), Hibernate(Session)에서 엔티티 상태와 변경사항을 관리하는 1차 캐시 역할

엔티티를 조회하면 해당 엔티티는 영속성 컨텍스트에 등록되어 엔티티 식별자를 키로 하는 Map에 저장

같은 트랜잭션 내에서 동일 엔티티를 여러 번 조회해도, 영속성 컨텍스트에서 캐싱된 객체를 반환 → 캐시를 조회하기 떄문에 빠름

## EntityManager & Session

JPA의 EntityManager와 Hibernate의 Session은 영속성 컨텍스트의 API 역할을 하며, 기능적으로 거의 동일 → Hibernate 5.2부터 Session이 EntityManager를 확장하여 두 인터페이스의 경계가 더욱 모호

## 엔티티 상태 전이

- Tansient/New : 새로 생성, 아직 DB에 저장되지 않은 상태
- Managed : persist() 호출 등으로 영속성 컨텍스트에 등록되어 관리되는 상태
- Detached : 영속성 컨텍스트에서 분리된 상태 (evict, close 등)
- Removed : remove() 호출로 삭제 예약
    - persist() : INSERT 예약
    - remove() : DELETE 예약
    - 엔티티 필드 변경 (Managed 상태) : UPDATE 예약

![https://ultrakain.gitbooks.io/jpa/content/chapter3/chapter3.3.html](/springboot/img/image.png)

https://ultrakain.gitbooks.io/jpa/content/chapter3/chapter3.3.html

## Write-Behind Cache & Flushing

영속성 컨텍스트는 트랜잭션 write-behind 캐시로 동작

- 엔티티 변경 (INSERT/UPDATE/DELETE)은 즉시 DB에 반영되지 않고, 일단 영속성 컨텍스트에 저장
- flush 시점에만 실제 DML(INSERT/UPDATE/DELETE)이 DB에 반영

flush 시, 영속성 컨텍스트는 엔티티가 로드된 이후 변경되었는지 감지하여 UPDATE 쿼리 생성 → 이 행위를 Dirty Checking이라고 함

## Flushing

flush()는 Persistence Context와 DB를 동기화하는 작업

- 트랜잭션 commit 시 변경사항을 안전하게 저장
- 쿼리 실행 전, in-memory 변경사항이 쿼리 결과에 반영 (read-your-writes 일관성)

수동( flush() ), 자동(FlushMode) 모두 지원

**JPA FlushModeType은 두 가지**

- AUTO(default) : 쿼리 실행 전, 트랜잭션 커밋 전 자동 flush
- COMMIT : 트랜잭션 커밋 시에만 flush → 쿼리 실행 전에는 변경사항이 쿼리 결과에 반영되지 않을 수 있음

**Hibernate FlushMode는 네 가지**

- AUTO : 트랜잭션 커밋 시 flush → 쿼리 실행 전에는 반드시 flush하지 않음
- ALWAYS : 모든 쿼리 실행 전 + 커밋 시 flush
- COMMIT : 커밋 시에만 flush
- MANUAL : 수동 flush만 가능(자동 flush 없음)
