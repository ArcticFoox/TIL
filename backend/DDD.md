# 도메인 주도 설계(DDD, Domain-Driven Design)
## DDD가 나오게 된 배경

기존 전통적 구조:

```
Controller
 └ Service
    └ Repository
       └ Entity (거의 DTO 역할)
```

문제점:

* Service가 점점 **God Object**가 됨
* 비즈니스 규칙이 Service, Util, Validator에 흩어짐
* Entity는 setter만 가득한 **빈 껍데기**
* 도메인 변경 시 영향 범위 예측 불가

DDD는 이 문제를 이렇게 정의합니다:

> “소프트웨어 복잡성의 원인은 기술이 아니라 **도메인 이해 부족**이다.”


## DDD의 핵심 철학

> **비즈니스 도메인을 중심에 두고, 코드 구조가 도메인 언어를 그대로 드러내도록 설계하자**


## 핵심 개념 정리

### Ubiquitous Language (보편 언어)

* **개발자 + 기획자 + 도메인 전문가가 동일한 용어 사용**
* 코드, 메서드명, 패키지명에 그대로 반영

잘못된 예

```java
processData()
handleLogic()
doBusiness()
```

DDD식 예

```java
placeOrder()
cancelOrder()
expirePayment()
```

> "이 메서드는 무슨 일을 하는지 **비즈니스 용어로 바로 설명 가능해야 함**


### Bounded Context

* 하나의 도메인 용어가 **문맥마다 의미가 달라질 수 있음**
* 그래서 **도메인 경계를 명확히 분리**

예:

* `Order`

    * 주문 도메인: 구매 행위
    * 정산 도메인: 정산 대상
    * 배송 도메인: 배송 단위

→ 같은 `Order`라도 **같은 Entity면 안 됨**

```
order-domain
settlement-domain
delivery-domain
```

Spring 패키지 구조로 보면:

```
order
 ├─ domain
 ├─ application
 ├─ infrastructure
```


### Entity vs Value Object

#### Entity

* **식별자(ID)** 가 있음
* 생명주기 관리 대상

```java
class Order {
    private Long id;
    private OrderStatus status;
}
```

#### Value Object

* **값 자체가 의미**
* 불변(immutable)
* equals/hashCode 중요

```java
class Money {
    private final BigDecimal amount;
    private final Currency currency;
}
```

### Aggregate & Aggregate Root

* 변경의 **일관성 경계**
* 외부에서는 **Aggregate Root만 접근 가능**

```
Order (Aggregate Root)
 ├─ OrderLine
 ├─ ShippingInfo
```

규칙:

* OrderLine을 직접 저장/수정 불가
* 반드시 Order를 통해 변경 

```java
order.changeShippingAddress(newAddress);
```

→ **트랜잭션 경계와 거의 1:1**


### Domain Service

Entity에 넣기 애매한 **도메인 규칙**

```java
class DiscountService {
    Money calculate(Order order, Member member);
}
```

* 상태 없음
* “행위”만 존재



### Application Service (유스케이스)

* 트랜잭션 제어
* 도메인 객체 조합
* 기술적 책임 담당

```java
@Transactional
public void placeOrder(PlaceOrderCommand cmd) {
    Order order = orderFactory.create(cmd);
    orderRepository.save(order);
}
```

> **비즈니스 규칙은 Application Service에 두지 않는다**


## 4. 전통적인 Service vs DDD Service 차이

| 항목         | 전통적 방식   | DDD          |
| ---------- | -------- | ------------ |
| Service 역할 | 모든 로직 처리 | 유스케이스 조율     |
| Entity     | 데이터 덩어리  | 규칙의 주체       |
| 트랜잭션       | 로직 단위    | Aggregate 단위 |
| 변경 영향      | 큼        | 경계 내로 제한     |


### 절충안

* 핵심 도메인만 DDD 적용
* CRUD 위주 도메인은 전통 방식 유지
* CQRS 혼합 사용

---

### JPA와 DDD의 충돌 지점

#### 문제 1: JPA 엔티티 = 도메인 엔티티?

* JPA 제약(@Entity, 프록시, 기본 생성자)

해결:

* **도메인 모델 중심**
* JPA는 persistence detail로 취급

#### 문제 2: Lazy Loading → 도메인 오염

* Application Service에서 필요한 데이터만 로딩
* Aggregate 크기 최소화


## 트레이드오프

### 단점

* 러닝 커브 큼
* 초기 개발 속도 느림
* 팀 내 도메인 이해도 요구 높음
* 패키지만 DDD, 로직은 빈약
* Entity에 getter만 가득
* Service 폭증

### 사용하지 않는 경우

* 단순 CRUD
* 요구사항 거의 안 변함
* 도메인 복잡도 낮음
