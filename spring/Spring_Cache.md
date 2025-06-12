# Spring Cache

스프링에서 스프링 AOP 기반으로 캐시가 작동하며 어노테이션으로 AOP를 설정할 수 있어 간편하게 사용 가능

@Cacheable, @CachePut, @CacheEvict 을 사용하면 쉽게 캐시 가능

## @Cacheable

캐시 생성 및 전달 담당

캐시에 데이터가 없을 경우 기존의 로직을 실행 후 캐시에 데이터를 추가하고, 캐시에 데이터가 있으면 반환

```java
  // 캐시 저장 (Key 미 지정)
  @Cacheable("memberCacheStore")
  public Member cacheable(String date) {
    System.out.println("cacheable 실행");
    ...
    return member;
  }

  // 캐시 저장 (Key 지정)
  @Cacheable(value = "memberCacheStore", key = "#member.name")
  public Member cacheableByKey(Member member) {
    System.out.println("cacheable 실행");
    ...
    return member;
  }

  // 조건부 캐시 저장 (With Condition)
  @Cacheable(value = "memberCacheStore", key = "#member.name", condition = "#member.name.length() > 5")
  public Member cacheableWithCondition(Member member) {
    System.out.println("cacheableWithCondition 실행");
    ...
    return member;
  }
```

### Key 미 지정 시

date 값이 처음 임력되면 메서드 실행

cacheable 실행 문자열 출력

다시 실행 시 문자열 출력 안됨

### Key 지정 시

member.name 을 기준으로 유무를 판단

### 조건부 캐싱

@Cacheable 어노테이션에 condition 속성을 통해 적용 가능

member 객체의 name 필드 값의 길이가 5를 초과하는 경우에만 캐싱

## @CachePut

캐시 내용 수정 담당

@Cacheable과 유사하게 실행 결과를 캐시에 저장하지만,

조회 시 저장된 캐시 내용을 사용하지 않고 항상 메서드를 실행

```java
// 캐시 저장 (Cacheable과 유사하게 실행 결과를 캐시에 저장하지만, 조회 시에 저장된 캐시 내용을 사용하지는 않고, 항상 메소드의 로직을 실행한다는 점에서 다르다.)
  @CachePut(value = "memberCacheStore", key = "#member.name")
  public Member cachePut(Member member) {
    System.out.println("cachePut 실행");
    ...
    return member;
  }
```

메서드 실행에는 영향을 주지 않기에,

매번 chchePut 실행 이 출력되며 member 객체에 데이터가 캐싱됨

## @CacheEvict

캐시 삭제 담당

캐시 이름을 넣으면 메소드가 실행될 때 캐시의 내용 제거

기본적으로 메소드의 키에 해당하는 캐시만 제거

```java

  @CacheEvict("memberCacheStore")
  public Member cacheEvict(String date) {
    System.out.println("cacheEvict 실행");
    ...
    return null;
  }

  // key
  @CacheEvict(value = "memberCacheStore", key = "#member.name")
  public Member cacheEvictByKey(Member member) {
    System.out.println("cacheEvictByKey 실행");
    ...
    return member;
  }

  // allEntries
  @CacheEvict(value = "memberCacheStore", allEntries = true)
  public Member cacheEvictAllEntries() {
    System.out.println("cacheEvictAllEntries 실행");
    ...
    return null;
  }

  // beforeInvocation 
  @CacheEvict(value = "memberCacheStore", beforeInvocation = true)
  public Member cacheEvictBeforeInvocation() {
    System.out.println("cacheEvictBeforeInvocation 실행");
    ...
    return null;
  }
```

### 기본

key에 해당하는 date 값에 대한 member 객체 데이터가 캐싱되어 있을 경우,

해당 키에 대한 요청이 들어오면 메서드가 실행되며 저장된 캐시 삭제

### allEntries

캐시에 저장된 값을 모두 제거

### beforeInvocation

메서드 실행 이전에 캐시를 제거하도록 지정

출처

[https://hyeri0903.tistory.com/237](https://hyeri0903.tistory.com/237)