# Tomcat

아파치 소프트웨어 재단의 웹 어플리케이션 서버(Web Application Server, WAS)

자바 서블릿을 실행시키고 JSP코드가 포함되어 있는 동적 웹 페이지를 구동시켜주는 역할

내장되어 있는 웹 서버를 이용해 독립적으로 사용될 수도 있으나 아파치(https), Nginx 등의 웹 서버와 함께 사용 가능

## 구조

Coyote(HTTP Component): Tomcat에 TCP를 통한 프로토콜 지원

Catalina(Servlet Container): 자바 서블릿을 호스팅하는 환경

Jasper(JSP Engine): 실제 JSP 페이지의 요청을 처리하는 Servlet

![image.png](/backend/img/image.png)

### 동작

HTTP 요청을 Coyote에서 받아서 Catalina로 전달

Catalina에서 전달받은 HTTP 요청을 처리할 웹 어플리케이션을 찾고 WEB-INF/web.xml 파일 내용을 참조하여 요청을 전달

요청된 Servlet을 통해 생성된 JSP 파일들이 호출될 때 Jasper가 Validation Check / Compile 등을 수행

Tomcat은 JVM 위에서 동작

하나의 JVM에서 하나의 Tomcat Instance가 하나의 Process로 동작

하나의 Server에는 여러 개의 Service가 존재 가능, 각각의 Service는 1개의 Engine과 여러 개의 Connector로 구성

Engine은 Catalina Servlet Engine이라고도 불리며, 정의된 Connector로 들어온 요청을 하위에 있는 해당 Host에게 전달해주는 역할을 수행

하나의 Engine에는 여러 개의 Host가 존재 가능, Host는 가상호스트 이름을 나타내며, 호스트 이름이 곧 url에 매핑

Host에는 여러 개의 Context가 존재 가능, Context는 하나의 Web Application을 나타내며 주로 ~.war 파일의 형태로 배포

Tomcat Server가 요청을 받으면, Catalina(Tomcat Engine)가 요청에 맞는 Context(Context path)를 찾고, Context는 자신이 설정된 어플리케이션의 deployment descripotr file(web.xml)을 기반으로 전달받은 요청을 서블릿에게 전달하여 처리

## 파일구조

- bin : 톰캣실행에 필요한 실행, 종료 스크립트 파일이 위치
- conf : server.xml 및 서버 전체 설정과 관련한 톰캣 설정파일들이 위치
- lib : 아파치와 같은 다른 웹서버와 톰캣을 연결해주는 바이너리 모듈들이 포함되어있고 톰캣구동하는데 필요한 라이브러리들이 위치
- logs : 톰캣실행 로그파일 위치
- temp : 톰캣이 실행되는 동안 임시파일이 위치
- webapps : 웹어플리케이션 위치
- work : jsp파일을 서블릿형태로 변환한 java파일과 class 파일을 저장하는 위치

## Embedded Tomcat

톰캣은 기본적으로 Java로 개발

웹 어플리케이션에 내장시켜서 어플리케이션과 동일한 JVM에서 실행 가능

마이크로서비스에 적합

Springboot에서는 기본적으로 Embedded Tomcat이 포함되어 있음

### Tomcat과의 차이점

virtual host가 지원되지 않음

WAS 설정을 웹 어플리케이션 내부에서 해야함(Java Code, application.properties 등)