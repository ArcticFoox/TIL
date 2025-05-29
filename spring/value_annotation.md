# @Value

application.properties, application.yml 설정 파일에 설정한 값을 주입할 수 있는 어노테이션

## 주의점

### 주입 시점

대상 컴포넌트가 스프링 빈으로 등록되고 의존 관계를 주입할 때 동장

환경 변수를 주입받는 대상 클래스에 @Component 어노테이션을 붙여주지 않는다면,

해당 클래스는 컴포턴트 스캔의 대상이 되지 않아 스프링 빈으로 등록되지 않고, 

@Value 어노테이션 또한 동작하지 않음

### 주입 방식

필드 주입, 생성자 주입, setter 주입 등 방법을 목적에 맞게 사용해야 함

### 프로퍼티 파일의 경로와 스코프

application.yml 파일이 클래스 패스에 존재해야 하고,

프로퍼티 파일이 여러 개의 경우 우선순위를 고려해야 함

@configurationProperties 어노테이션과의 차이점

스프링의 프로퍼티 파일의 값은 Environment에 등록

두 어노테이션 모두 이 값을 불러올 수 있지만,

@Value의 경우에는 단일 값을 주입받기 위해 사용되며, RelaxedBinding이 적용되지 않음

→ RelaxedBingding은 프로퍼티 이름이 조금 달라도 유연하게 바인딩 시켜주는 규칙

@ConfigurationProperties의 경우 한번에 여러 값을 바인딩 받을 수 있으며, RelaxedBinding 적용