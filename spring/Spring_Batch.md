# Spring Batch

대용량 배치 처리에 최적화된 프레임워크

## 핵심 개념

| 구성 요소 | 설명 |
| --- | --- |
| **Job** | 배치 작업 단위. 하나의 배치 실행 단위입니다. |
| **Step** | Job을 구성하는 단위 작업. 여러 Step으로 Job을 구성할 수 있습니다. |
| **JobInstance** | Job이 실행된 "논리적 인스턴스". 파라미터가 다르면 다른 JobInstance로 간주합니다. |
| **JobExecution** | JobInstance의 실행 기록. 성공, 실패 여부를 포함합니다. |
| **StepExecution** | Step 하나에 대한 실행 기록. 상태, 처리 건수, 예외 등 포함. |

## 기본 처리 흐름

```java
Job
 └─ Step 1 ──> Step 2 ──> Step 3 (성공/실패 조건 따라 흐름 변경 가능)
```

### 내부 구조

Step은 보통 아래 구성으로 이루어짐

```java
ItemReader → ItemProcessor → ItemWriter
```

ItemReader: 데이터를 읽음

ItemProcessor: 데이터를 가공, 검증, 필터링

ItemWriter: 데이터를 저장

## 주요 기능 및 특징

| 기능 | 설명 |
| --- | --- |
| 트랜잭션 관리 | Step 단위, Chunk 단위 트랜잭션 관리 |
| 재시도/스킵 | 예외 발생 시 재시도 또는 스킵 처리 가능 |
| 체크포인트 (Chunk 기반) | 일정 단위로 Commit, 중간 저장 가능 |
| 병렬 처리 | 멀티 스레드, 파티셔닝, 그리드 처리 지원 |
| 상태 저장 | Job/Step 실행 결과를 DB에 저장하여 실패 후 재시작 가능 |
| 스케줄링 | Spring Scheduler, Quartz 등과 연동 가능 |