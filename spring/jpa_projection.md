# Spring Data JPA Projection
## Projection 이란?
엔티티 전체를 조회하지 않고,  
필요한 컬럼만 조회해서 반환하는 방식

## 이점
- 불필요한 엔티티 로딩 제거
- N + 1 문제 예방
- API 응답 모델과 엔티티 분리
- Batch / Quartz 환경에서 메모리 절약

## Entity 반환의 위험성
```java
order.getA().getB().getC();
```
- 연관관계가 LAZY 라면 N+1 발생
- 트랜잭션 밖에서는 `LazyInitializationException` 발생
- API 응답 시 엔티티 노출 위험
- 필요 없는 컬럼까지 전부 조회

## 예시
```java
public interface OrderSummary {
    Long getId();
    LocalDateTime getCreatedAt();
    BigDecimal getTotalAmount();
}
```
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    @Query("""
        SELECT o.id AS id,
               o.createdAt AS createdAt,
               o.totalAmount AS totalAmount
        FROM Order o
        WHERE o.createdAt > :lastChecked
        ORDER BY o.createdAt ASC
    """)
    List<OrderSummary> findOrderSummaries(@Param("lastChecked") LocalDateTime lastChecked);
}
```
**특징**
- Spring Data가 프록시 객체 생성
- 필요한 컬럼만 조회
- DTO 클래스 불필요

