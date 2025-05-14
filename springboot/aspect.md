# Aspect

```java
@Slf4j
@Aspect
@Component
public class AspectExample 
    @Around("execution(* hello.aop.test..*(..))") // AspectJ 표현식
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); // join point 시그니처
        return joinPoint.proceed(); // 실제 타깃 호출
    }
}
```

@Aspect의 어노테이션이 붙은 클래스를 Advisor(어드바이저)라고 하고,

메서드는 Advice, @Around는 Pointcut으로 Advice를 적용할 대상 지정

@Around 어드바이스를 사용할 경우 메서드의 파라미터로 “ProceedingJoinPoint”를 꼭 넣어줘야함

proceed()는 다음 어드바이스나 타겟을 호출하는 것으로, 어드바이스를 사용하기 위해서 꼭 proceed() 메서드 호출

```java
@Slf4j
@Aspect
@Component
public class AspectExample {
 
    // hello.aop.test 패키지와 하위 패키지에 적용
    @Pointcut("execution(* hello.aop.test..*(..))")
    private void allTestLog() {} //  포인트컷 시그니쳐 
 
    @Around("allTestLog()") // 포인트컷을 메서드로 만들어 사용
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); // join point 시그니처
        return joinPoint.proceed(); // 실제 타깃 호출
    }
}
```

1. @Pointcut에 포인트컷 사용.
2. 메서드 이름과 파라미터를 합쳐서 포인트컷 시그니처라고 한다
3. 메서드의 반환 타입은 void 이어야 함
4. 코드 내용은 비워둔다.
5. 어드바이스에 포인트컷을 직접 지정해도 되지만, 포인트컷 시그니처도 사용 가능

## 어드바이스 종류

- @Around : **메서드 호출 전후에 수행, 가장 강력한 어드바이스**, 조인 포인트 실행 여부 선택, 반환 값 변환, 예외 변환 등이 가능 (사실 이거 하나만 사용해도 무방)
- @Before : 조인 포인트 실행 이전에 실행
- @AfterReturning : 조인 포인트가 정상 완료후 실행
- @AfterThrowing : 메서드가 예외를 던지는 경우 실행
- @After : 조인 포인트가 정상 또는 예외에 관계없이 실행(finally)

출처

[https://hstory0208.tistory.com/entry/Spring-스프링-AOP-Aspect-사용법](https://hstory0208.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-AOP-Aspect-%EC%82%AC%EC%9A%A9%EB%B2%95)