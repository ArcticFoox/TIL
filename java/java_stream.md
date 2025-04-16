# Java Stream

일련의 데이터를 함수형 연산을 통해 표준화된 방법으로 쉽게 가공, 처리할 수 있다.

대량의 데이터를 표준화된 방식으로 처리하기 위한 용도

### 가독성 향상

‘선언적’코딩 가능

연속적으로 필터링, 매핑, 정렬을 ‘체이닝’하여 표현 가능

### 병렬처리 지원

데이터의 흐름을 나누어서 멀티 스레드를 활용해 병렬로 처리하고 처리 후 합치는 과정을 통해 대량의 데이터를 빠르고 쉽게 처리할 수 있다는 장점 보유

간단하게 parallel() 또는 parallelStream() 연산을 추가하는 것만으로 병렬처리 가능

## Java Stream 구조와 특징

생성 → 가공 → 소비 구조로 구성

### 생성

Stream API를 사용하여 가공하기 위해 최초 1번 수행

모든 데이터가 한꺼번에 메모리에 로드되지 않고 필요할 때만 로드

### 가공

필터(filter), 변형(map), 정렬(sort) 등의 가공

연속으로 수행 가능(체이닝)

### 최종 연산

수행 후 Stream API로 중간 연산이나 최종 연산을 다시 처리할 수 없음

연산 처리 시 특징으로 ‘지연평가(Lasy Evaluation)’ 사용

<aside>
💡

> 지연평가(Lasy Evaluation)
> 

중간 연산들을 연산을 호출할 때 즉시 수행되지 않고, 최종 연산이 호출될 때까지 지연

</aside>

## 사용법

### 생성

데이터의 집합인 컬렉션(배열, ArrayList, Set, Map…)으로 생성

Stream에서 지원하는 generate() 메소드나 iterate() 메소드를 이용하여 다양한 형태의 데이터 Stream 생성 가능

 

### 가공

**filter**

```java
// 짝수만 걸러내는 연산
IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);
```

**map**

```java
IntStream.rangeClosed(1, 100).map(n -> n * n);
```

**sorted**

```java
IntStream.rangeClosed(1, 100).sorted();
```

**peak**

```java
// 나머지 값 출력
IntStream.rangeClosed(1, 100).map(n -> n * n).peak(n -> sout(n));
```

**distinct**

```java
// 1부터 100까지의 숫자를 10으로 나눈 나머지 값 중에서 중복을 제거
IntStream.rangeClosed(1, 100).map(n -> n % 10).distinct()
```

**limit**

```java
// 1부터 100까지의 숫자 중에서 처음 10개의 숫자만 사용
IntStream.rangeClosed(1, 100).limit(10);
```

### 소비

**요소의 출력**

forEach() - stream의 각 요소를 순회하면서 출력 등의 처리를 위해 사용

**요소의 소모**

reduce() - stream의 요소를 줄여나가면서 연산 수행

처음 두 요소를 가지고 연산한 결과를 다음 요소와 연산해서 최종적인 값을 구하기 위해 사용

**요소의 검색**

findFirst(), findAny() - 특정 조건에 맞는 요소를 찾기 위해 사용

findFirst()는 시퀀셜처리에 사용

findAny()는 병렬처리에 사용

**요소의 검사**

anyMatch(), allMatch(), noneMatch() - 조건에 맞는지 확인을 위해 사용

**요소의 통계**

count(), min(), max() - 요소의 개수, 최소값, 최대값을 구하기 위해 사용

**요소의 연산**

sum(), average() - 합계, 평균을 구하기 위해 사용

**요소의 수집**

collect() - stream의 요소를 수집하여 원하는 형태로 변환하기 위해 사용

Java Collector 인터페이스를 매개변수로 호출

Java에서 제공하는 Collectors 클래스에서 이미 만들어둔 method를 이용해서 요소를 변환하여 사용