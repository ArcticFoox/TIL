# @Async

spring에서 제공하는 Thread Pool을 활용하는 비동기 메소드 지원 Annotation

기존 Java에서 비동기 방식으로 메서드를 구현 할 땐 아래와 같이 구현했다.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class GillogAsync {

    static ExecutorService executorService = Executors.newFixedThreadPool(5);

    public void asyncMethod(final String message) throws Exception {
        executorService.submit(new Runnable() {
            @Override
            public void run() {
                // do something
            }            
        });
    }
}
```

`java.util.concurrent.ExecutorService`를 활용해서 method 정의 때 마다 Runnable의 run()을 재구현해야 하는 등 동일한 작업들의 반복이 잦았다.

# @Async

간단하게 사용하려면, 단순히 Application Class에 @EnableAsync 추가 및 method @Async 추가를 통해 사용 가능

기본 설정으로 SimpleAsyncTaskExecutor 사용

# AsyncConfigurerSupport

```java
import java.util.concurrent.Executor;

import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurerSupport;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(30);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("DDAJA-ASYNC-");
        executor.initialize();
        return executor;
    }
}
```

- @Configuration : Spring 설정 관련 Class로 @Component 등록되어 Scanning 될 수 있음
- @EnableAsync : Spring method에서 비동기 기능을 사용가능하게 활성화
- CorePoolSize : 기본 실행 대기하는 Thread의 수
- MaxPoolSIze : 동시 동작하는 최대 Thread 수
- QueueCapacity : MaxPoolSize 초과 요청에서 Thread 생성 요청 시, 최대 수용 가능한 Queue 수
- ThreadNamePrefix : 생성되는 Thread 접두사 지정

# 주의사항

Private method는 사용 불가

self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가

QueueCapacity 초과 요청에 대한 비동기 method 호출시 방어 코드 작성