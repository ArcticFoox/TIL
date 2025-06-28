# ForkJoinWorkerThread

## 전체 아키텍처

```java
                    +-------------------------------+
                    |       ForkJoinPool            |
                    |  - WorkQueues[] (스레드 큐)     |
                    |  - Submission queue          |
                    +-------------------------------+
                              ▲
    ┌────────────────────┐    │    ┌──────────────────────┐
    │ 외부에서 작업 제출      │────┼───▶│ WorkQueue (Submitter)│
    └────────────────────┘    │    └──────────────────────┘
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
           ┌────────────────┐  ┌────────────────┐
           │ ForkJoinWorker │  │ ForkJoinWorker │  
           │    Thread A    │  │    Thread B    │
           └────────────────┘  └────────────────┘

 각 워커는 자신의 작업 큐를 가지며,
 큐가 비면 다른 워커의 큐에서 stealing 시도
```

## 생성 시점

```java
// JDK 내부
static final ForkJoinPool common = new ForkJoinPool(
    parallelism,       // 기본: CPU 수 - 1
    commonThreadFactory,
    null,              // UncaughtExceptionHandler
    true               // asyncMode
);
```

parallelism: Runtime.getRuntime().availableProcessors() - 1

ForkJoinWorkerThreadFactory: commonThreadFactory가 워커 생성 시 ForkJoinWorkerThread를 생성

asyncMode = true: FIFO 방식 큐

## 워커 스레드 초기화

ForkJoinWorkerThreadFactory.newThread(…) 호출 → ForkJoinWorkerThread 생성

ForkJoinWorkerThread는 ForkJoinPool.registerWorker()를 통해 자신만의 WorkQueue 등록

```java
public ForkJoinWorkerThread(ForkJoinPool pool) {
    super("ForkJoinPool.commonPool-worker-" + ...);
    this.pool = pool;
    this.workQueue = pool.registerWorker(this);  // 자신의 큐 등록
}
```

생성 직후부터 풀에 참여할 준비를 마침

## 작업 제출

```java
CompletableFuture.supplyAsync(() -> "Hello");
```

→ 내부적으로 ForkJoinTask.adapt(Runnable/Callable) → ForkJoinPool.commonPool().execute(task)

제출된 작업은 Submitter WorkQueue에 저장

ForkJoinPool.scan(…)을 통해 워커가 큐를 돌며 작업을 가져감

## 작업 실행

각 워커 스레드는 다음을 반복

```java
while (!shutdown) {
    // 1. 내 큐에서 작업 꺼냄 (pop)
    // 2. 없으면 work stealing 시도
    // 3. 계속 없으면 block
}
```

내부 루프에서는 WorkQueue.poll(), WorkQueue.pop(), WorkQueue.pollAt() 등으로 작업 획득

ForkJonTask.doExec() 호출 → 작업 실행

## Work Stealing

자신의 큐가 비었을 경우, 다른 워커의 큐에서 FIFO로 작업을 훔침

스케줄링 메커니즘은 경량화되어 있어 락을 거의 사용하지 않음(CAS 등 원자 연산 기반)

```java
// 예시: 다른 워커 큐에서 도난 시도
ForkJoinTask<?> t = victimQueue.poll();  // FIFO 방식
```

이를 통해 부하를 동적으로 균형 조절

## 예외 처리

ForkJoinTask.invoke(), ForkJoinTask.get() 등은 내부 예외를 포착하고 ExecutionException으로 감쌈

워커 스레드에서 발생한 RuntimeException은 기본적으로 무시되거나 로깅됨

UncaughtExceptionHandler를 ForkJoinPool 생성 시 등록할 수 있음

## 재사용 및 생명주기

ForkJoinPool.commonPool()의 스레드는 JVM 종료까지 살아 있음

기본적으로 shutdown되지 않으며, 풀 내에서 작업이 없으면 idle 상태로 대기

자바가 종료되지 않으면 GC 대상이 아님

재사용되므로 ThreadLocal, MDC, SecurityContext 등은 명시적 정리 필요

## 결론

```java
[작업 제출]
    ↓
[ForkJoinPool.commonPool()]
    ↓
[Submitter 큐에 등록]
    ↓
[워커 스레드 생성 → WorkQueue 등록]
    ↓
[자기 큐에서 작업 pop → 실행]
    ↓ (큐 비었을 경우)
[다른 큐에서 작업 stealing → 실행]
    ↓
[스레드는 idle 상태로 재사용]
```