저번에 쓴 내용을 잊어 다시 작성
# Logback

log4j 이후 출시된 log4j 항상된 Java 로깅 라이브러리

slf4j의 구현체로써 Springboot의 기본 log로 사용되고 있으며,

spring-boot-starter-web안 spring-boot-starter-logging의 logback이 기본적으로 포함되어 있음

# Level

Trace : 가장 상세한 로그 레벨, 코드의 흐름을 따라가며 디버깅 정보를 기록

Debug : 디버깅을 위한 로그 레벨, 프로그램의 상태 및 실행 중에 발생하는 중요한 이벤트 기록

Info : 일반적인 정보를 기록하는 로그 레벨, 프로그램의 주요 이벤트 및 상태 변경을 기록

Warning : 예외적인 상황을 기록하는 로그 레벨, 잠재적인 문제 또는 예상치 못한 동작을 알림

Error : 심각한 에러를 기록하는 로그 레벨, 예외 상황 또는 잘못된 동작을 나타냄

Fatal : 가장 심각한 로그 레벨, 치명적인 오류를 기록하고 프로그램 중단 또는 비정상 종료를 알림

# Logback 설정파일 읽기 순서

Classpath(일반적으로 resources 디렉토리 밑) 내에서 logback.xml 파일 탐색

- 보통 Maven, Gradle 등의 빌드 도구를 사용할 경우, 소스 코드와 함께 리소스 디렉토리(src/main/resources)에 logback.xml 파일을 배치
- 만약 logback.xml 파일이 클래스 경로에 있는 경우, logback은 해당 파일을 자동으로 로드
- 스프링 부트 애플리케이션의 경우, ****logback-spring.xml이 먼저 로딩
    - logback-spring.xml은 스프링 부트의 자동 구성 기능을 활용하여 로딩되는 특별한 설정 파일
    - 스프링 부트에서는 logback-spring.xml을 사용하여 일부 기본 설정을 제공하고, 프로파일링과 관련된 환경 변수 등을 처리하는 기능을 제공
    - logback-spring.xml은 logback.xml 파일보다 우선순위가 높음
- logback-spring.xml 이 없다면 .yml(.properties) 파일 의 설정을 읽음
    - logback-spring.xml과 .yml(.properties)파일이 동시에 있으면 .yml(.properties) 파일을 적용한 후 `.xml` 파일이 적용

Classpath 내에서 logback-test.xml 파일 탐색

- ogback-test.xml 파일은 logback.xml 파일보다 우선순위가 높음
- 보통 테스트 환경에서 사용되며, 같은 디렉토리(src/test/resources)에 배치

없다면 logback.groovy 파일 탐색

- logback.groovy 파일은 logback.xml 파일보다 우선순위가 높음
- 마찬가지로 클래스 경로 또는 테스트 환경 디렉토리에 배치될 수 있음

Configuration 속성을 사용하여 logback 설정을 지정

- 만약 위의 파일들을 찾을 수 없는 경우, logback은 Configuration 속성에 지정된 설정 파일 경로로부터 설정을 로드
- Configuration 속성은 Java 시스템 프로퍼티나 환경 변수를 통해 설정될 수 있음

기본 설정을 사용

- 모든 설정 파일을 찾지 못하면, logback은 기본 설정을 사용
- 기본 설정은 로그를 콘솔에 출력하고, 출력 형식이 간단한 형태

# Appender

로그를 출력할 위치, 출력 형식 등을 설정하는 구성요소

### ConsoleAppender

로그를 OutputStream에 write하여, 최종적으로 콘솔에 출력되도록 함

### FileAppender

로그의 내용을 지정된 file에 기록

### RollingFileAppender

FileAppender로부터 상속받은 Appender로써, 날짜, 최대 용량 등을 설정하여 지정한 파일명 패턴에 따라 로그가 다른 파일에 기록되도록 함

Logback-Core의 기본 Appender 외에도 Logback-Classic 모듈의 다양한 Appender를 사용하여 로그를 원격위치에 기록 가능

Appender들의 하위 항목으로 출력 형식(Layout Pattern)을 지정하여 각 Appender마다 원하는 내용을 출력 가능

ex) %logger(Logger 이름), %thread(현재 스레드명), %level(로그 레벨), %msg(로그메시지), %n(new line) 등

→ 주의할 점으로 ‘파일이름.%d{yyyy-MM-dd}.%i.’ 같이 date 형식과 %i 가 존재하지 않으면 에러 발생

### SMTPAppender

로그를 메일에 찍어 보냄

### DBAppender

DB에 로그를 찍음

출처:

[https://0soo.tistory.com/242](https://0soo.tistory.com/242)