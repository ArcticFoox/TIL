# AutoConfiguration

시작은 @SpringBootApplication 안 @EnableAutoConfiguration 에서 일어남

@EnableAutoConfiguration은 @Import(AutoConfigurationImportSelector.class)를 통해 자동 구성 클래스를 가져 옴

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

자동 구성 클래스를 가져올 때는 AutoConfigurationImportSelector 클래스의 selectImports(AnnotationMetadata annotationMetadata)라는 메서드를 이용하고,

getAutoConfigurationEntry(AnnotationMetadata annotationMetadata); 메서드를 통해 Import할 클래스가 무엇인지 알 수 있게 됨

### 동작과정

1. getCandidateConfigurations(annotationMetadata, attributes); - AutoConfiguration의 후보들을 가져온다.
2. removeDuplicates(configurations); - 중복을 제거한다.
3. getExclusions(annotationMetadata, attributes); - 자동 설정에서 제외되는 설정에 대한 정보를 가져온다.
4. configurations.removeAll(exclusions); - 제외되는 설정을 제거한다.
5. getConfigurationClassFilter().filter(configurations); - 필터를 적용한다.

출처

[https://velog.io/@realsy/Spring-Boot-AutoConfiguration-동작-원리](https://velog.io/@realsy/Spring-Boot-AutoConfiguration-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)