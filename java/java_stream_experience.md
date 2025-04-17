# Java Stream exprience

Map<String, List<CustomObject>> target 객체 안에서,

CustomInfo.T 의 중복을 해결해야 하는 과제가 생겼다.

Stream을 이용해 중복 제거가 된 map을 만들 계획이다.

결과부터 설명하자면 다음과 같다.

```java
Map<String, List<CustomObject>> result = target.entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        entry -> entry.getValue().stream()
            .collect(Collectors.collectingAndThen(
                Collectors.toMap(
                    CustomInfo::getT,
                    Function.identity(),
                    (existing, replacement) -> existing
                ),
                map -> new ArrayList<>(map.values())
            ))
    ));
```

단계별로 뜯어보자.

1. `target.entrySet().stream()` 
    - target은 `Map<String, List<CustomObject>>` 타입
    - `entrySet()`을 호출하면 `Set<Map.Entry<String, List<CustomObject>>>`가 반환
    - 반환된 것에 `stream()`으로 스트림 처리 준비
    
2. `.collect(Collectors.toMap(…))` 
    - 스트림의 각 entry를 가공해서 다시 `Map<String, List<CustomObject>>`로 만듦
    
    ```java
    Collectors.toMap(
        Map.Entry::getKey, // key: 원래의 key 유지
        entry -> entry.getValue().stream() // value: 중복 제거된 List<CustomInfo>
    )
    ```
    
3. `.collect(Collectors.collectingAndThen(…))` 
    - collectingAndThen()은 두 개의 인자를 받아
        - 먼저 `Collectors.toMap(…)`을 수행
        - 결과로 나온 Map을 후처리로 List로 바꿈
    
    ```java
    .collect(Collectors.collectingAndThen(
        Collectors.toMap(...), map -> new ArrayList<>(map.values())
    ))
    ```
    

1. `Collectors.toMap(…)`
    
    ```java
    Collectors.toMap(
        CustomInfo::getT,              // key: CustomObject의 t 값
        Function.identity(),             // value: 원본 객체 그대로
        (existing, replacement) -> existing  // 중복 발생 시 기존 값 유지
    )
    ```
    
    - `CustomInfo::getT` → 각 객체의 T 값을 기준으로 key를 만듦
    - `Function.identity()` → 각 객체 그대로 저장
    - (existing, replacement) → existing → 같은 Tid가 여러 번 나오면 첫 번째 것을 유지(중복 제거)

여기서 궁금증이 생겼다.

처음엔 `filter().distinct()`와의 차이점은 무엇일까?

예제를 확인해보자

```java
List<CustomObject> list = List.of(
    new CustomObject("apple", 1),
    new CustomObject("banana", 2),
    new CustomObject("apple", 3)
);
```

`list.stream().distinct()` → equals()와 hashCode() 기준

apple(1)과 apple(3)은 key는 같아도 객체가 다르고, equals()를 오버라이드하지 않았다면 다른 객체로 간주

```java
list.stream()
    .filter(obj -> obj.getValue() > 0)
    .distinct()
```

`list.stream().filter(…).distinct()` → filter()는 조건 필터링이지만 중복 제거는 여전히 전체 객체 기준

equals()와 hashCode() 오버라이드 했을 경우

```java
public class CustomObject {
    private String t;
    private int value;

    // 생성자, getter, setter 등 생략

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof CustomObject)) return false;
        CustomObject that = (CustomObject) o;
        return Objects.equals(t, that.t);  // t 기준만 비교
    }

    @Override
    public int hashCode() {
        return Objects.hash(t);  // 역시 t 기준
    }
}

List<CustomObject> list = List.of(
    new CustomObject("apple", 1),
    new CustomObject("banana", 2),
    new CustomObject("apple", 3)
);

[CustomObject("apple", 1), CustomObject("banana", 2)]
```

- 위에서 equals()/hashCode()가 T 기준으로 오버라이드하여 distinct() 적용시 중복 제거

`Collectors.toMap(CustomObject::getTid, Function.identity(), …)`

- 여기선 Tid 값만 기준으로 중복 제거
- 즉, ‘apple’이라는 Tid 값을 가진 객체는 하나만 남김
- 어떤 걸 남길지는 merge 함수로 제어 가능 ( (a, b) → b )

| 조건 | equals() 오버라이드 여부 | 중복 제거 방식 | 제어 가능성 |
| --- | --- | --- | --- |
| distinct() | 필요(equals, hashCode가 t 기준이어야 작동) | set을 사용해 제거 | 남길 객체 선택 불가 |
| Collectors.toMap(…) | 불필요 | t를 key로 사용 → 자동 중복 제거 | merge 함수로 둘 중 선택 가능 |