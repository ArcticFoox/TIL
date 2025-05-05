# Logback

log4j 아키텍처 기반으로 재작성된 logger 사용, 또한 SLF4J 지원하기에 언제든지 다른 logger로 전환 가능

## 특징

- SiftingAppender가 로그 파일을 특정 주제별로 분류하여 HTTP Session별 파일 저장, 사용자별 별도 파일 저장 등 가능
- Exception 발생 시 참조했던 외부 라이브러리 버전까지 출력
- 별도의 삭제 스케줄러 설정 및 개발 필요없이 maxHistory 설정을 통해 주기적으로 Archive 파일을 자동 삭제 가능
- 어플리케이션을 중지 시켰다가 재가동해도 서버 중지 없이 이전 시점부터 복구 지원
- 로그 레벨 변경시 내부 스캐닝하는 별도의 쓰레드를 보유하여 서버 재기동 할 필요 없음

## 작성법

### 순서

1. ERROR: 요청을 처리하는 중 오류가 발생한 경우 표시한다.
2. WARN: 처리 가능한 문제, 향후 시스템 에러의 원인이 될 수 있는 경고성 메세지를 나타낸다.
3. INFO: 상태변경과 같은 정보성 로그를 표시한다.
4. DEBUG: 프로그램을 디버깅하기 위한 정보를 표시한다.
5. TRACE: 추적 레벨은 Debug보다 훨씬 상세한 정보를 나타낸다.

### Pattern

- %logger{length} : Logger name을 축약 할 수 있다. {length}는 최대 글자 수 ex)logger{35}
- %-5level : 로그 레벨, -5는 출력의 고정폭 값(5글자)
- %msg : - 로그 메세지(=%message)
- ${PID:-} : 프로세스 아이디
- %d : 로그 기록시간
- %p : 로깅 레벨
- %F : 로깅이 발생한 프로그램 파일명
- %M : 로깅일 발생한 메소드의 명
- %I : 로깅이 발생한 호출지의 정보
- %L : 로깅이 발생한 호출지의 라인 수
- %thread : 현재 Thread 명
- %t : 로깅이 발생한 Threrad명
- %c : 로깅이 발생한 카테고리
- %C : 로깅이 발생한 클래스 명
- %m : 로그 메세지
- %n : 줄바꿈
- %% : %를 출력
- %r : 애플리케이션 시작 이후 부터 로깅이 발생한 시점 까지의 시간(ms)

### logback-spring.xml

appender와 logger 두 개로 구분

appender는 log의 형태 설정, logger는 설정한 appender를 참조하여 package와 level 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds">

    <!-- 속성 또는 변수 설정 -->
    <property resource="application.properties"/>
    <property name="${name}" value="${value}"/>

    <!-- application.properties 설정값을 가져올때 사용 -->
    <springProperty name="${name}" source="spring.datasource.driverClassName"/>
    <springProperty scope="context" name="${name}" source="logging.level.root"/>

    <!-- 어떤 속성의 appender를 사용할지 클레스 및 이름 설정 -->
    <appender name="${name}" class="${appender class}">
        <!-- 각 append class에 맞는 설정값이 다름.
             ch.qos.logback.core.ConsoleAppender,
            ch.qos.logback.core.FileAppender,
            ch.qos.logback.core.rolling.RollingFileAppender,
            ch.qos.logback.classic.db.DBAppender,
            ch.qos.logback.classic.net.SMTPAppender 등
        -->
    </appender>

    <!-- 전체 로그 출력설정 -->
    <root level="${proerty}">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </root>

    <!-- 패키지 별 로그 출력설정 -->
    <logger name="org.hibernate" level="debug">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </logger>
    <!-- 해당 패키지 하위 로그를 출력하고 싶지 않을때 additivity="false" -->
    <logger name="org.springframework" level="debug" additivity="false">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </logger>

</configuration>
```

- `ch.qos.logback.core.ConsoleAppender` : 콘솔에 로그 출력
    
    ```xml
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%level] [%logger] : %msg%n</pattern>
        </encoder>
    </appender>
    ```
    
- `ch.qos.logback.core.FileAppender` : 파일에 로그 저장(최대 보관 일 수 등을 지정)
    
    ```xml
    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>log-${bySecond}.txt</file>
        <append>true</append>
        <!-- set immediateFlush to false for much higher logging throughput -->
        <immediateFlush>true</immediateFlush>
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    ```
    
- `ch.qos.logback.core.rolling.RollingFileAppender` : 여러개의 파일을 순회하면서 로그 출력
- `ch.qos.logback.classic.net.SMTPAppender` : 로그를 메일로 보냄
    
    ```xml
    <appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
        <smtpHost>ADDRESS-OF-YOUR-SMTP-HOST</smtpHost>
        <to>EMAIL-DESTINATION</to>
        <to>ANOTHER_EMAIL_DESTINATION</to> <!-- additional destinations are possible -->
        <from>SENDER-EMAIL</from>
        <subject>TESTING: %logger{20} - %m</subject>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%date %-5level %logger{35} - %message%n</pattern>
        </layout>       
    </appender>
    ```
    
- `ch.qos.logback.classic.db.DBAppender` : DB에 로그를 쌓는다.
    
    ```xml
    <property resource="application.properties" />
    <springProperty name="spring.datasource.driverClassName" source="spring.datasource.driverClassName"/>
    <springProperty name="spring.datasource.url" source="spring.datasource.url"/>
    <springProperty name="spring.datasource.username" source="spring.datasource.username"/>
    <springProperty name="spring.datasource.password" source="spring.datasource.password"/>
    
    <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
            <driverClass>${spring.datasource.driverClassName}</driverClass>
            <url>${spring.datasource.url}</url>
            <user>${spring.datasource.username}</user>
            <password>${spring.datasource.password}</password>
        </connectionSource>
    </appender>
    ```
    
    DBAppender 사용 시 Table 생성을 요구함
    
    ```sql
    DROP TABLE IF EXISTS logging_event_property;
    DROP TABLE IF EXISTS logging_event_exception;
    DROP TABLE IF EXISTS logging_event;
    
    CREATE TABLE logging_event
    (
        timestmp         BIGINT NOT NULL,
        formatted_message  TEXT NOT NULL,
        logger_name       VARCHAR(254) NOT NULL,
        level_string      VARCHAR(254) NOT NULL,
        thread_name       VARCHAR(254),
        reference_flag    SMALLINT,
        arg0              VARCHAR(254),
        arg1              VARCHAR(254),
        arg2              VARCHAR(254),
        arg3              VARCHAR(254),
        caller_filename   VARCHAR(254) NOT NULL,
        caller_class      VARCHAR(254) NOT NULL,
        caller_method     VARCHAR(254) NOT NULL,
        caller_line       CHAR(4) NOT NULL,
        event_id          BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY
    );
    
    CREATE TABLE logging_event_property
    (
        event_id          BIGINT NOT NULL,
        mapped_key        VARCHAR(254) NOT NULL,
        mapped_value      TEXT,
        PRIMARY KEY(event_id, mapped_key),
        FOREIGN KEY (event_id) REFERENCES logging_event(event_id)
    );
    
    CREATE TABLE logging_event_exception
    (
        event_id         BIGINT NOT NULL,
        i                SMALLINT NOT NULL,
        trace_line       VARCHAR(254) NOT NULL,
        PRIMARY KEY(event_id, i),
        FOREIGN KEY (event_id) REFERENCES logging_event(event_id)
    );
    ```
    

출처

[https://agileryuhaeul.tistory.com/entry/Logback-이란](https://agileryuhaeul.tistory.com/entry/Logback-%EC%9D%B4%EB%9E%80)