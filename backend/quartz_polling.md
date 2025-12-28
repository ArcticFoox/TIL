# quartz polling
## architecture overview
```scss
Quartz Trigger (예: 30초, 1분)
        ↓
Quartz Job
        ↓
ChangeDetectorService
        ↓
MariaDB 조회
        ↓
이전 상태와 비교
        ↓
변화 감지 시
        ↓
NotificationService (Slack / Email / Webhook 등)
```

## 핵심 로직
created_at > last_checked_at 인 레코드를 조회하여 변화 감지

### Quartz Polling 전략
```sql
last_checked_at = 마지막으로 처리한 created_at
SELECT * FROM orders WHERE created_at > last_checked_at
```
```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("""
        SELECT o FROM Order o
        WHERE o.createdAt > :lastChecked
        ORDER BY o.createdAt ASC
    """)
    List<Order> findNewOrders(@Param("lastChecked") LocalDateTime lastChecked);
}
```
### 동시성 & 중복 알림 방지
동일 created_at 발생 않도록 복합 키 사용
```sql
ALTER TABLE polling_state
ADD COLUMN last_created_at DATETIME,
ADD COLUMN last_order_id BIGINT;
```
```java
@Query("""
    SELECT o FROM Order o
    WHERE (o.createdAt > :createdAt)
       OR (o.createdAt = :createdAt AND o.id > :lastId)
    ORDER BY o.createdAt ASC, o.id ASC
""")
List<Order> findNewOrdersSafely(
    @Param("createdAt") LocalDateTime createdAt,
    @Param("lastId") Long lastId
);
```
