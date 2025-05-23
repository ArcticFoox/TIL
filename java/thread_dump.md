# Thread dump

웹 서버에서는 동시 사용자를 처리하기 위해 많은 수의 스레드를 사용

두 개 이상의 스레드가 같은 자원을 이용할 때는 필연적으로 스레드 간의 경합이 발생하고 경우에 따라서는 데드락 발생 가능성 있음

스레드는 다른 스레드와 동시 실행이 가능하므로 여러 스레드가 공유 자원을 사용할 때 정합성을 보장하려면 스레드 동기화로 한 번에 하나의 스레드만 공유 자원에 접근 해야 함

Java에서는 Monitor를 이용해 스레드를 동기화 하여, 모든 Java 객체는 하나의 Monitor를 가짐

Monitor는 하나의 스레드만 소유할 수 있고 어떠한 스레드가 소유한 Monitor를 다른 스레드가 획득하려면 해당 Monitor를 소유하고 있는 스레드가 Monitor를 해제할 때까지 Wait Queue에서 대기

**Thread dump는 프로세스에 속한 모든 thread들의 상태를 기록한 것이며, 발생한 문제들을 진단, 분석하고 jvm 성능 최적화에 필요한 정보 제공**

![dump1.png](/java/img/dump1.png)

- NEW: 스레드가 생성되었지만 아직 실행되지 않은 상태
- RUNNABLE: 현재 CPU를 점유하고 작업을 수행 중인 상태이며 운영체제의 자원 분배로 인해 WAITING 상태가 될 수도 있다.
- BLOCKED: Monitor를 획득하기 위해 다른 스레드가 락을 해제하기를 기다리는 상태
- WAITING: wait() 메서드, join() 메서드, park() 메서드 등을 이용해 대기하고 있는 상태
- TIMED_WAITING: WAITING과 같지만, 정해진 시간만 대기
- TERMINATED: 종료 상태

## 분석

jstack을 통해 분석 가능

```java
// pid 획득
$ jps -v
// 아래와 같이 획득도 가능하다
$ ps -ef | grep java

// 획득한 pid를 이용하여 treadump.txt로 덤프 내용 저장
// -l 옵션을 주면 잠금 세부 사항도 확인할 수 있다.
$ jstack -l [PID] > threadump.txt
```

![dump2.png](/java/img/dump2.png)

- Thread Name
    - 스레드 이름이며, 이름을 변경하여 사용하는 경우 스레드 덤프에도 반영된다. 일반적으로 스레드 덤프를 해석하기 쉽게 의미 있는 이름으로 설정하는 것이 좋다.
- ID
    - JVM 내 각 스레드에 할당된 고유 ID이다.
- Thread Priority
    - Java 스레드의 우선순위이다.
- OS Thread Priority
    - 자바의 스레드는 운영체제(OS)의 스레드와 매핑이 되는데, 매핑된 운영체제 스레드의 우선순위이다.
- Java Level Thread ID
    - JVM 내부(JNI 코드)에서 관리하는 Native Thread 구조체의 포인터 주소이다.
- Native Thread ID
    - 자바 스레드에 매핑된 OS 스레드의 ID이다.
    - Window에서는 OS Level의 스레드 ID이며, Linux에서는 LWP(Light Weight Process)의 ID를 의미한다.
- Thread State
    - 스레드의 상태이다.
- Last Known Java Stack Pointer
    - 스레드의 현재 Stack Pointer(SP)의 주소를 의미한다.
- Call Stack
    - 현재 스레드가 수행되는 함수들의 호출 관계(콜 스택)를 표현한다.