# Reduce Spring boot Test 
## 개요
스프링부트가 제공하는 테스트 어노테이션들은 모두 어플리케이션 컨텍스트를 만들어 조건에 맞는 빈을 찾아 등록
어플리케이션 설정을 잘못하면 불필요한 테스트 비용 발생
- 특정 기능을 위한 @EnableXXX 어노테이션
- 빈 탐색을 위한 @ComponentScan

## @EnableXXX
스프링부트에서 테스트를 위한 어플리케이션 컨텍스트를 만들 때 설정의 기준이 되는 클래스는 @SpringBootApplication이 붙은 클래스
일반적으로 해당 어노테이션은 메인 클래스에 존재
때문에 <b>메인 클래스에는 특정한 기능을 위한 설정을 추가하지 않는 것이 좋음</b>
```java
@SpringBootApplication
@EnableBatchProcessing
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

@WebMvcTest, @DataJpaTest 등 특정 기능을 위한 테스트 어노테이션들은 @EnableXXX 어노테이션을 포함  
불필요하게 테스트를 무겁게 만드므로 특정 기능을 위한 설정 클래스를 만들어 분리하는 것이 좋음
```java
@Configuration
@EnableFeignClients("com.example.myapp")
class OpenFeignConfig {

}
```
위처럼 설정 클래스를 분리하여 만드는 것이 좋음

## @ComponentScan
@ComponentScan은 특정 패키지를 기준으로 빈을 탐색
테스트 대상이 되는 클래스가 속한 패키지와 그 하위 패키지를 스캔
따라서 테스트 대상이 되는 클래스가 속한 패키지에 불필요한 빈이 많다면 테스트가 무거워짐
```java
@SpringBootApplication
@ComponentScan({ "com.example.app", "com.example.another" })
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
    
}
```
만약 설정 클래스로 뺄 수 없다면 테스트 패키지에 별도의 @SpringBootConfiguration을 만들거나,
테스트에 대한 소스 위치를 지정하여 기본 설정을 비활성화

출처
https://mangkyu.tistory.com/243