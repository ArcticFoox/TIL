# Jackson 3 with Spring Boot 4
Jackson은 JVM 환경에서 가장 널리 사용되는 JSON 처리 라이브러리  
## 개선점
- Jackson 2와 Jackson 3을 동시에 사용할 수 있도록 개선
- JDK 17 이상에서만 지원
- Spring의 JSON View 기본 동작과 Jackson을 일관되게 맞춤
- 논블로킹 파서 개선
- JsonNode API에서 null-safety 정교화
- JsonWriteFeature 기본값 재조정
- @JsonCreator 사용 필요성이 증가한 문제 개선

## 변경점
### 패키지 변경
기존 : com.fasterxml.jackson.*  
변경 : tools.jackson  

### 기본 설정 변경
Jackson 3의 기본값은 Jackson 2와 다름  
속성 정렬 기본값 : MapperFeature.SORT_PROPERTIES_ALPHABETICALLY = true  
날짜 직렬화 : DateTimeFeature.WRITE_DATES_AS_TIMESTAMPS = false  

### Jackson 모듈 처리 방식 변화
Jackson 2의 여러 모듈(parameter-names, datatype-jsr310 등)이 Jackson 3에 기본 포함  
또한 Spring은 이제 JDK Service Loader 기반으로 모듈을 자동 탐색  
필요한 경우 `JsonMapper.Builder`를 통해 직접 설정 가능

### ObjectMapper -> JsonMapper
Jackson 3의 핵심 변화 중 하나가 불변 기반 API 도입
Jackson 2: mutable ObjectMapper
Jackson 3: immutable JsonMapper (ObjectMapper 상속)

### JsonView
Jackson 2와 Spring Framework 6에서는 `JsonView`를 사용하기 위해 `MappingJacksonValue`를 감싸야 했음  
```java
var jacksonValue = new MappingJacksonValue(user);
jacksonValue.setSerializationView(Summary.class);
```
Spring Framework 7 부터는 `JacksonJsonHttpMessageConverter`가 `SmartHttpMessageConverter`를 구현하여 힌트 기반 처리 방식 지원  
```java
var response = this.restClient.post().uri("http://localhost:8080/create")
    .hint(JsonView.class.getName(), Summary.class).body(user)
    .retrieve().body(String.class);
```

## Spring Data 4에서의 Jackson 3 지원
### Spring Data Commons

Jackson 3 기본 사용, Jackson 2는 클래스패스 감지 시 폴백  
JacksonResourceReader, JacksonRepositoryPopulatorFactoryBean 추가  
XML namespace 기반 Jackson 2 사용 시 Java Config로 대체 필요  

### Spring Data Redis

JacksonHashMapper, JacksonJsonRedisSerializer 등 Jackson 3 기반 버전 제공  
기존 Jackson2ObjectReader/Writer는 명칭 정리 필요  

### Spring Data REST

Jackson 2 모드 지원 제거  
Spring HATEOAS도 동일하게 Jackson 3 필수  

Spring Data Couchbase, Elasticsearch, Drivers  

내부적으로 Jackson 2를 쓰는 경우가 있으나 엔티티 직렬화에는 영향 없음  


출처: https://digitalbourgeois.tistory.com/2402 [평범한 직장인이 사는 세상:티스토리]