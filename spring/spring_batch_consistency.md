# Spring Batch 정합성 보장 방법

Spring Batch에서 **정합성(Consistency)** 이란  
배치 실패, 재시작, 병렬 실행 상황에서도 데이터가 **중복 없이**, **누락 없이**, **의도한 상태로만** 변경되도록 보장하는 것을 의미한다.


## 1. 트랜잭션 경계 설계 (기본이자 핵심)

### Chunk 기반 트랜잭션
```java
.stepBuilderFactory.get("step")
    .<Input, Output>chunk(100)
````

* 하나의 Chunk는 하나의 트랜잭션 단위
* Chunk 내부의 `read → process → write` 는 원자적으로 처리됨
* 실패 시 해당 Chunk 전체 rollback
* 이전 Chunk는 commit 유지

### Chunk 크기 트레이드오프

| Chunk 크기 | 장점       | 단점                |
| -------- | -------- | ----------------- |
| 작음       | 롤백 범위 작음 | DB 트랜잭션 수 증가      |
| 큼        | 처리 성능 향상 | 롤백 비용 증가, 락 장기 점유 |

> 데이터 변경량이 크거나 락 경합이 민감한 경우 Chunk를 작게 설정

---

## 2. JobRepository 기반 재시작 정합성

Spring Batch는 메타데이터 기반으로 재시작 정합성을 보장한다.

### JobRepository가 관리하는 정보

* JobInstance 중복 실행 방지
* Job / Step 실행 상태 관리
* 마지막 commit 지점부터 재시작 가능

관련 테이블:

```text
BATCH_JOB_INSTANCE
BATCH_JOB_EXECUTION
BATCH_STEP_EXECUTION
```

### 재시작 정합성을 깨는 사례

```sql
SELECT * FROM table LIMIT 100;
```

### 올바른 Reader 예시

```sql
WHERE id > :lastId
ORDER BY id ASC
```

> Reader는 항상 동일한 순서와 기준을 유지해야 한다.



## 3. Idempotency(멱등성) 보장

같은 데이터를 여러 번 처리해도 결과는 한 번 처리한 것과 같아야 한다.

### DB 레벨 멱등성 (가장 강력)

* UNIQUE KEY
* UPSERT / MERGE / ON DUPLICATE KEY

```sql
INSERT INTO result (id, value)
VALUES (?, ?)
ON DUPLICATE KEY UPDATE value = value;
```

### 처리 여부 컬럼 방식

```sql
processed_yn CHAR(1),
processed_at DATETIME
```

Reader 조건:

```sql
WHERE processed_yn = 'N'
```

### 트레이드오프

* 상태 컬럼 방식: 락 경합 가능성
* UNIQUE KEY 방식: 예외 처리 필수

> Batch는 "한 번만 처리"가 아니라 "여러 번 처리해도 안전"해야 한다.



## 4. 동시 실행(Race Condition) 방지

### Job 수준 중복 실행 방지

```java
.incrementer(new RunIdIncrementer())
```

### Step 수준 DB 락

```sql
SELECT ... FOR UPDATE;
```

* 동일 데이터 동시 처리 방지
* 대량 처리 시 성능 저하 가능

### 분산 환경 (멀티 인스턴스)

* Redis 또는 DB 기반 Lock 사용
* 반드시 TTL 설정 필요

---

## 5. Skip / Retry 정책과 정합성

### Skip 설정

```java
.skip(Exception.class)
.skipLimit(100)
```

* Skip된 데이터는 실패 상태로 남음
* 의도된 누락이므로 사후 관리 필요

### Retry 설정

```java
.retry(DeadlockLoserDataAccessException.class)
.retryLimit(3)
```

* DB Deadlock, 일시적 네트워크 오류 대응에 필수

### 주의 사항

* Retry 대상 로직은 반드시 멱등성 보장
* 외부 API 호출은 중복 호출 위험 존재



## 6. 병렬 처리 시 정합성 전략

### Partitioning (권장)

* 데이터 범위를 명확히 분할

```text
ID 1 ~ 1000
ID 1001 ~ 2000
```

* 중복 처리 발생하지 않음
* 가장 안전한 병렬 처리 방식

### Multi-threaded Step (주의)

```java
.taskExecutor(new SimpleAsyncTaskExecutor())
```

* Reader 공유로 정합성 문제 발생 가능
* 고급 설정 없으면 실무 비권장


## 7. 외부 시스템 연동 시 정합성

### 문제 상황

* DB 커밋은 성공
* 외부 시스템 연동 실패

### 해결 패턴: Outbox Pattern

1. DB에 이벤트 데이터 저장
2. 별도 배치 또는 컨슈머가 외부 시스템 전송
3. 성공 시 전송 완료 상태로 변경

> 트랜잭션 경계를 분리하여 정합성을 보장하는 대표 패턴


## 8. 정합성 보장을 위한 실무 체크리스트

* Chunk 크기를 근거 있게 설정했는가
* Reader가 재시작 가능 구조인가
* Writer가 멱등성을 보장하는가
* 중복 실행 방지 장치가 있는가
* 병렬 처리 시 데이터 분할 기준이 명확한가
* 실패 및 재시작 시나리오를 테스트했는가
