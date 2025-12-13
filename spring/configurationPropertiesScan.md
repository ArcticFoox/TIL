# ConfigurationProertiesScan Annotation in Spring
## @EnableConfigurationProperties의 한계점
@EnableConfigurationProperties 어노테이션은 특정 클래스에 대해 ConfigurationProperties를 활성화하는 데 사용됩니다. 그러나 이 어노테이션을 사용할 때는 활성화할 클래스들을 명시적으로 지정해야 하는 한계점이 있음

즉, 새로운 설정 클래스가 추가될 때마다 해당 클래스를 어노테이션에 추가해야 함

## @ConfigurationPropertiesScan 사용법
패키지를 기반으로 @ConfigurationProperties가 등록된 클래스들을 찾아 값들을 주입하고 빈으로 등록
@Component나 그 하위 어노테이션(@Configuration 등)이 붙은 클래스들은 스캔되지 않는다.