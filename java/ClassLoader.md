# ClassLoader

자바는 동적 로드, 즉 컴파일 타임이 아니라 런타임(바이트 코드를 실행할 때)에 클래스를 링크하고 로드하는 특징을 가짐

런타임에 동적으로 클래스를 로딩한다는 것은 JVM이 미리 모든 클래스에 대한 정보를 메소드 영역에 로딩하지 않는다는 것을 의미

정리하자면, 클래스 로더는 런타임 중에 JVM의 메소드 영역에 동적으로 Java 클래스를 로드하는 역할

![image.png](/java/img/classloader.png)

클래스 로더 내에는 Loading, Linking, Initalization 과정이 있음

## Loading

자바 바이트 코드(.class)를 메소드 영역에 저장

각 자바 바이트 코드(.class)는 JVM에 의해 메소드 영역에 다음 정보를 저장

- 로드된 클래스를 비롯한 그의 부모 클래스의 정보
- 클래스 파일과 Class, Interface, Enum의 관련 여부
- 변수나 메소드 등의 정보

## Linking

검증(verify): 읽어들인 클래스가 자바 언어 명세 및 JVM 명세에 명시된대로 잘 구성되어 있는지 검사

준비(prepare): 클래스가 필요로 하는 메모리 할당, 클래스에 정의된 필드, 메소드, 인터페이스를 나타내는 데이터 구조를 준비

분석(resolve): 심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체

## Initialization

클래스 변수들(static 변수)을 적절한 값으로 초기화

# 클래스 로더 종류

## 부트스트랩 클래스 로더(Bootstrap Class Loader)

JVM 시작 시 가장 최초로 실행되는 클래스 로더

자바 클래스를 로드하는 것이 아닌 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와,

최소한의 자바 클래스(Object, Class, ClassLoader, util.* 등)들을 로드

Java8: jre/lib/rt.jar 및 기타 핵심 라이브러리와 같은 JDK 내부 클래스 로드

Java9 이후: ClassLoader 내 최상위 클래스들만 로드

## 확장 클래스 로더(Extension Class Loader)

부트스트랩 클래스 로더를 부모로 갖는 클래스 로더로서,

확장 자바 클래스들을 로드

java.ext.dirs 환경 변수에 설정된 디렉토리의 클래스 파일을 로드하고,

이 값이 설정되어 있지 않는 경우 ${JAVA_HOME}/jre/lib/ext 에 있는 클래스 파일을 로드

java 8: URLClassLoader를 상속하며, jre/lib/ext 내 모든 클래스 로드

java 9 이후: Platform Loader로 변경되었으며, BuiltinClassLoader를 상속. Inner Static 클래스로 구현

java 11: [다음 클래스](https://docs.oracle.com/en/java/javase/11/docs/api/index.html) 로드

## 애플리케이션 클래스 로더(Application Class Loader)

자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드

우리가 만든 .class 확장자 파일을 로드

# 클래스로더 동작 방식

JVM의 클래스 로더는 새로운 클래스를 로드해야할 때, 다음과 같이 동작

- `JVM`의 `Method Area`에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
- `Method Area`에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
- `시스템 클래스 로더`는 확장 클래스 로더에 요청을 위임한다.
- `확장 클래스 로더`는 부트스트랩 클래스 로더에 요청을 위임합니다.
- `부트스트랩 클래스로더`는 부트스트랩 Classpath(JDK/JRE/LIB) 에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 `확장 클래스로더`에게 요청을 넘긴다.
- `확장 클래스 로더`는 확장 Classpath(JDK/JRE/LIB/EXT) 에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 `시스템 클래스 로더`에게 요청을 넘긴다.
- `애플리케이션 클래스로더`는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 ClassNotFoundException을 발생시킨다.