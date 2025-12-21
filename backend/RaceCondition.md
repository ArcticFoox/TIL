# RaceCondition
여러 스레드가 동시에 실행될 때, 특정 코드나 데이터에 접근 순서가 비결정적으로 실행되어,  
예상치 못한 결과를 초래하는 상황을 의미

## 발생 이유
- 공유 자원의 존재
- 동시 실행 발생
- 동기화가 없음

### 자주 발생하는 상황
싱글톤 빈의 mutable 필드
```java
@Service
class UserService {
    private int temp; // 위험
}
```
CompletableFuture + shared object
```java
CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    sharedObject.setValue("Value from Future 1");
});
CompletableFuture<Void> future2 = CompletableFuture.runAsync(() -> {
    sharedObject.setValue("Value from Future 2");
});
```
캐시 구현(Map)  
HashMap X ConcurrentHashMap O
  
DB 레벨
- 두 트랜잭션이 같은 row 수정
- optimistic/pessimistic lock 필요

