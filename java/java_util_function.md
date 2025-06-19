# java.util.function

# 함수적 인터페이스

오직 하나의 추상 메서드만 가진 인터페이스

@FunctionalInterface 어노테이션으로 명시 가능

람다식으로 간결하게 표현 가능

```java
@FunctionalInterface
public interface MyFunction {
    void run();
}
```

# 주요 인터페이스

## Function<T, R>

하나의 입력 T를 받아서 결과 R을 반환하는 함수

데이터를 변환(transform)할 때 주로 사용

### 메서드

R apply(T t): 입력 값을 받아 결과 반환

andThen(Function<R, V> after): 함수 조합(먼저 현재 함수 실행, 이후 after 실행)

compose(Function<V, T> before): 함수 조합(먼저 before 실행, 이후 현재 함수 실행)

```java
Function<String, Integer> stringLength = s -> s.length();
System.out.println(stringLength.apply("hello")); // 5

Function<Integer, String> intToStr = i -> "Length is " + i;
Function<String, String> combined = stringLength.andThen(intToStr);
System.out.println(combined.apply("hello")); // Length is 5
```

## Consumer<T>

입력 값 T를 받아서 소비만 하고, 반환값 없음

주로 출력, 저장 등 부수 효과(side effect)를 발생시키는 데 사용

### 메서드

void accept(T t)

andThen(Consumer<T> after): 여러 Consumer 연결 실행

```java
Consumer<String> greeter = name -> System.out.println("Hello, " + name);
greeter.accept("Alice"); // Hello, Alice
```

## Supplier<T>

입력이 없고, 결과 `T`만 반환합니다.

주로 **데이터 생성**, **값 공급**에 사용됩니다.

### 메서드

T get()

```java
Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get()); // 0.123456...
```

## Predicate<T>

입력값 `T`에 대해 `true` 또는 `false`를 반환하는 조건 함수입니다.

**필터링**, **조건 검사** 등에 많이 사용됩니다.

### 메서드

`boolean test(T t)`

`and`, `or`, `negate` 등 조건 결합 가능

```java
Predicate<String> isEmpty = s -> s.isEmpty();
System.out.println(isEmpty.test("")); // true

Predicate<String> isNotEmpty = isEmpty.negate();
System.out.println(isNotEmpty.test("abc")); // true
```

## UnaryOperator<T>

입력과 출력 타입이 동일한 `Function<T, T>`입니다.

입력을 그대로 어떤 방식으로든 **가공**할 때 사용합니다.

### 메서드

`T apply(T t)`

```java
UnaryOperator<String> toUpper = s -> s.toUpperCase();
System.out.println(toUpper.apply("hello")); // HELLO
```

## BinaryOperator<T>

두 개의 동일한 타입 `T`를 받아, 같은 타입 `T`를 반환합니다.

두 값을 **결합(combine)**할 때 사용됩니다.

### 메서드

`T apply(T t1, T t2)`

```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
System.out.println(sum.apply(3, 5)); // 8
```

## BiFunction<T, U, R>

두 개의 입력값 `T`, `U`를 받아 하나의 결과 `R`을 반환합니다.

### 메서드

- `R apply(T t, U u)`

```java
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
System.out.println(repeat.apply("Hi", 3)); // HiHiHi
```

## BiConsumer<T, U>

두 개의 입력값 `T`, `U`를 받아 **작업만 수행**하고 결과는 반환하지 않습니다.

### 메서드

`void accept(T t, U u)`

```java
BiConsumer<String, Integer> printNTimes = (s, n) -> {
    for (int i = 0; i < n; i++) {
        System.out.println(s);
    }
};
printNTimes.accept("Hello", 2);
// Hello
// Hello
```

## BiPredicate<T, U>

두 개의 입력값 `T`, `U`에 대해 조건을 검사하고 `boolean`을 반환합니다.

### 메서드

`boolean test(T t, U u)`

```java
BiPredicate<String, String> equalsIgnoreCase = (a, b) -> a.equalsIgnoreCase(b);
System.out.println(equalsIgnoreCase.test("java", "JAVA")); // true
```