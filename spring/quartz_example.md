# Spring Batch + Quartz 사용 예시

## 1. 전체 아키텍처 개요

```

[ Quartz Scheduler ]
|
v
[ Quartz Job ]
|
v
[ Spring Batch JobLauncher ]
|
v
[ Spring Batch Job ]

````

- Quartz
  - Cron / Calendar / Misfire 처리
  - 다중 트리거, 클러스터링
- Spring Batch
  - 트랜잭션, 재시작, 정합성
  - Step 단위 실패 복구

---

## 2. 의존성 설정

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-batch'
    implementation 'org.springframework.boot:spring-boot-starter-quartz'
}
````


## 3. Spring Batch Job 정의

```java
@Configuration
@RequiredArgsConstructor
public class SampleBatchJobConfig {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager transactionManager;

    @Bean
    public Job sampleJob() {
        return new JobBuilder("sampleJob", jobRepository)
                .start(sampleStep())
                .build();
    }

    @Bean
    public Step sampleStep() {
        return new StepBuilder("sampleStep", jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Spring Batch Job 실행");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }
}
```


## 4. Quartz Job → Spring Batch 실행

Quartz Job은 **비즈니스 로직을 가지지 않고**
**JobLauncher만 호출**하는 것이 핵심 원칙

```java
@Component
@RequiredArgsConstructor
public class SampleQuartzJob extends QuartzJobBean {

    private final JobLauncher jobLauncher;
    private final Job sampleJob;

    @Override
    protected void executeInternal(JobExecutionContext context) {
        try {
            JobParameters jobParameters = new JobParametersBuilder()
                    .addLong("run.id", System.currentTimeMillis())
                    .toJobParameters();

            jobLauncher.run(sampleJob, jobParameters);

        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 5. Quartz Trigger 설정

### 5.1 JobDetail

```java
@Bean
public JobDetail sampleQuartzJobDetail() {
    return JobBuilder.newJob(SampleQuartzJob.class)
            .withIdentity("sampleQuartzJob")
            .storeDurably()
            .build();
}
```

### 5.2 Cron Trigger

```java
@Bean
public Trigger sampleQuartzTrigger() {
    return TriggerBuilder.newTrigger()
            .forJob(sampleQuartzJobDetail())
            .withIdentity("sampleQuartzTrigger")
            .withSchedule(
                CronScheduleBuilder.cronSchedule("0 0 2 * * ?")
            )
            .build();
}
```

## 6. Quartz + Batch 조합 시 주의사항

### 6.1 중복 실행 방지

* Quartz → 트리거 중복 가능
* Batch → `JobRepository` 기반 중복 실행 방지

```java
.addLong("run.id", System.currentTimeMillis())
```

또는

```java
.addString("businessDate", "2025-12-26")
```

### 6.2 동시 실행 제어

Quartz Job에 어노테이션 적용

```java
@DisallowConcurrentExecution
public class SampleQuartzJob extends QuartzJobBean {
}
```

→ 같은 JobDetail의 **동시 실행 방지**


### 6.3 실패 재시작 전략

| 책임     | 역할          |
| ------ | ----------- |
| Quartz | 언제 다시 실행할지  |
| Batch  | 어디서부터 재시작할지 |

* Step 실패 → `FAILED`
* 재실행 시 → `restartable=true`

## 7. 운영 환경에서의 일반적인 구성

```
[Quartz Cluster]
   - DB 기반 JobStore
   - Scheduler HA

[Spring Batch]
   - 단일 실행 보장
   - JobRepository 공유
```

### application.yml 예시

```yaml
spring:
  quartz:
    job-store-type: jdbc
    jdbc:
      initialize-schema: never
```