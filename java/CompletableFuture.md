# CompletableFuture

Java 8에서 도입된 java.util.concurrent 패키지의 클래스이며,

비동기 프로그래밍을 위해 사용

Future 인터페이스를 개선한 형태로,

논블로킹, 콜백 기반, 함수형 방식의 작업 흐름을 지원

## 주요 메서드

### supplyAsync/runAsync

비동기 작업 시작

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Hello";
});
```

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("비동기 작업 수행");
});
```

### thenApply, thenAccept, thenRun

thenApply: 결과 반환

thenAccept: 결과 소비

thenRun: 결과 필요 없이 후속 작업 실행

```java
// 결과를 가공
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(result -> result + " World");  

// 결과 출력
CompletableFuture<Void> printFuture = future.thenAccept(System.out::println);  

// 결과 없이 후처리
CompletableFuture<Void> run = future.thenRun(() -> System.out.println("완료됨")); 
```

### thenCombine, thenCompose

thenCombine: 두 작업의 결과를 조합

thenCompose: 두 작업을 순차로 연결 (flatten)

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = f1.thenCombine(f2, (a, b) -> a + " " + b); // Hello World
```

```java
CompletableFuture<String> composed = f1.thenCompose(str ->
    CompletableFuture.supplyAsync(() -> str + " World")); // Hello World
```

### exceptionally, handle

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("에러!");
    return "성공";
}).exceptionally(ex -> "예외 처리됨: " + ex.getMessage());
```

```java
future.handle((result, ex) -> {
    if (ex != null) return "예외 발생: " + ex.getMessage();
    else return "정상 결과: " + result;
});
```

### allOf,anyOf

allOf: 모든 작업 완료 후

anyOf: 하나라도 완료되면

```java
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2);
CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2);
```

## 내부 동작

내부적으로 ForkJoinPool.commonPool() 사용

→ Java 8부터 도입된 고성능 병렬 처리 프레임워크인 ForkJoinPool의 공용 인스턴스

이 공용 풀의 워커 스레드(ForkJoinWorkerThread)는 비동기 작업을 효율적으로 수행

### ForkJoinPool.commonPool() 구성

JVM에서 앱 시작시, 아래 방식으로 공용 풀이 한번 생성됨

```java
ForkJoinPool.commonPool();

//내부 구현
private static final ForkJoinPool common = new ForkJoinPool(
    Runtime.getRuntime().availableProcessors() - 1,
    commonPoolWorkerFactory,
    null,
    true // asyncMode
);
```

### 워커 스레드(ForkJoinWorkerThread)의 동작 흐름

- **스레드 풀 초기화**
    - 공용 풀이 생성되면 내부적으로 `ForkJoinWorkerThread` 객체를 풀에 등록합니다.
    - 각 스레드는 **자신만의 작업 큐**를 가짐.
- **작업 제출**
    - `CompletableFuture.runAsync(...)` 호출 시, 작업(Runnable)이 공용 풀에 제출됨.
    - 제출된 작업은 `ForkJoinTask` 형태로 변환되어 내부 큐에 저장됨.
- **작업 실행**
    - 스레드는 자신의 작업 큐에서 작업을 꺼내 실행.
    - 큐가 비면, 다른 스레드의 큐에서 작업을 훔쳐오는 **work-stealing**을 수행.
- **클래스 로더 관련**
    - `ForkJoinWorkerThread`는 생성 시 `contextClassLoader`를 명시적으로 설정하지 않음.
    - 즉, `Thread.currentThread().getContextClassLoader()` 호출 시 `null`일 수 있음 (특히 Docker나 커스텀 런타임 환경에서).
- **작업 종료 후**
    - 워커 스레드는 **풀링(pooling)** 되어 재사용됨.
    - GC가 되지 않고 계속 살아있어 추후 작업을 기다림.