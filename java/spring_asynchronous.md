# Spring Asynchronous

## CompletableFuture

runAsync()

```java
CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> System.out.println("Hello World!"));
cf.join();
```

Runnable 타입을 파라미터로 전달하기 때문에 어떤 결과 값을 담지 않음

supplyAsync()

```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() ->  "Hello World!");
System.out.println(cf.join());
```

supplier 타입을 넘기기 때문에 반환 값이 존재

get()

Future 인터페이스에 정의된 메소드로 checked exception인 InterruptedException과 ExecutionException을 던지므로 예외 처리 로직 필요

join()

CompletableFuture에 정의되어 있으며, checked exception을 발생시키지 않는 대신 unchecked CompletionException이 발생

### 작업 콜백

thenApply()

함수형 인터페이스 Function 타입을 파라미터로 받으며, 반환 값을 받아서 다른 값을 반환해주는 콜백이다.

```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return "hello world!";
}).thenApply(s -> {
    return s.toUpperCase();
});
System.out.println(cf.join()); // HELLO WORLD!
```

thenAccept()

함수형 인터페이스 Consumer를 파라미터로 받으며, 반환 값을 받아 처리하고 값을 반환하지 않는 콜백

```java
CompletableFuture<Void> cf = CompletableFuture.supplyAsync(() -> {
		return "hello world!";
}).thenAccept(System.out::println);

cf.join(); // hello world!
```

thenRun()

함수형 인터페이스 Runnable을 파라미터로 받으며, 반환 값을 받지 않고 그냥 다른 작업을 처리하고 값을 반환하지 않는 콜백

```java
CompletableFuture<Void> cf = CompletableFuture.supplyAsync(() -> {
    return "hello world!";
}).thenRun(() -> System.out.println("hello coco world!")); 
cf.join(); // hello coco world!;
```

### 비동기 작업 콜백

thenApplyAsync()

앞선 계산의 결과를 콜백 함수로 전달된 Function을 별도의 스레드에서 비동기적으로 실행

```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return "hello world!";
}).thenApplyAsync(s -> {
    return s.toUpperCase();
});
System.out.println(cf.join()); // HELLO WORLD!
```

thenAcceptAsync()

앞선 계산의 결과를 콜백 함수로 전달된 Consumer를 별도의 스레드에서 비동기적으로 실행

```java
CompletableFuture<Void> cf = CompletableFuture.supplyAsync(() -> {
    return "hello world!";
}).thenAcceptAsync(System.out::println); 
cf.join(); // hello world!
```

thenRunAsync()

앞선 계산의 결과와 상관없이 주어진 작업을 별도의 스레드에서 비동기적으로 실행

```java
CompletableFuture<Void> cf = CompletableFuture.supplyAsync(() -> {
    return "hello world!";
}).thenRunAsync(() -> System.out.println("hello coco world!")); 
cf.join(); // hello coco world!
```