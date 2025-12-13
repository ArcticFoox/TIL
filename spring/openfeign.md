# OpenFeign
선언적(declarative) HTTP Client 도구로써, 외부 API 호출을 쉽게 할 수 있도록 도와줌

선언적인 이란 어노테이션 사용을 의미

인터페이스에 어노테이션들만 붙여 구현

```java
@FeignClient(name = "ExchangeRateOpenFeign", url = "${exchange.currency.api.uri}")
public interface ExchangeRateOpenFeign {

    @GetMapping
    ExchangeRateResponse call(
            @RequestHeader String apiKey,
            @RequestParam Currency source,
            @RequestParam Currency currencies);

}
```

## 장점
- 인터페이스와 어노테이션 기반으로 작성할 코드가 줄어듬
- 익숙한 Spring MVC 어노테이션으로 개발이 가능
- 다른 Spring Cloud 기술(Eureka, Ribbon 등)과 쉽게 통합

## 단점
- 기본 HTTP Client가 Http2를 지원하지 않음
- 공식적으로 Reactive 모델을 지원하지 않음
- 경우에 따라 애플리케이션이 뜰 때 초기화 에러가 발생할 수 있음
- 테스트 도구를 제공하지 않음

## 부가 기능 설정
### Timeout 설정
```yaml
feign:
  client:
    config:
      default:
        connect-timeout: 5000  # 연결 타임아웃 (밀리초)
        read-timeout: 5000     # 읽기 타임아웃 (밀리초)
```
### Retry 설정
기본적으로 Retryer.NEVER_RETRY를 등록하여 Retry를 시도하지 않음  

추가설정을 해야 Retry가 동작함  

Feign이 제공하는 Retryer는 IOException이 발생한 경우에만 처리됨  

이외의 경우 재시도가 필요하다면 Spring-Retry를 이용하거나 ErrorDecoder, Interceptor 등으로 직접 구현하는 방법을 사용해야 함
```java
@Configuration
@EnableFeignClients("com.test.openfeign")
class OpenFeignConfig {

    @Bean
    Retryer.Default retryer() {
        // 0.1초의 간격으로 시작해 최대 3초의 간격으로 점점 증가하며, 최대5번 재시도한다.
        return new Retryer.Default(100L, TimeUnit.SECONDS.toMillis(3L), 5);
    }
}
```
### Logging 설정
Logger의 이름은 전체 인터페이스 이름이며, Feign Client들마다 만들어짐
- NONE: 로깅 안함(기본값)
- BASIC: 요청 메서드와 URL, 응답 상태 코드와 실행 시간
- HEADERS: BASIC + 요청 및 응답 헤더
- FULL: HEADERS + 요청 및 응답 바디와 메타데이터
```java
Configuration
@EnableFeignClients("com.test.openfeign")
class OpenFeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```
Feign은 DEBUG 레벨에서만 로그를 남길 수 있음

출처  
https://mangkyu.tistory.com/278