# Spring Batch 도메인별 Job 구성 정리

## 1. 기본 결론

- **Job은 도메인 단위로 분리**하는 것이 가장 효율적
- **BatchConfig는 하나로 관리하지 않음**

공통 인프라 Config

* 도메인별 Job Config
* 도메인별 Step Config

## 2. 왜 하나의 BatchConfig로 관리하면 안 되는가?

### 문제점

- 도메인 경계 붕괴 (User / Order / Settlement 로직 혼재)
- 변경 시 영향 범위 과도하게 증가
- 테스트 시 불필요한 Job 전부 로딩
- 장기적으로 모듈 분리·MSA 전환 난이도 상승

```java
@Configuration
public class BatchConfig { // 안 좋은 예
    @Bean Job userJob() { ... }
    @Bean Job orderJob() { ... }
}
```

## 3. 제안 아키텍처 구조

### 3.1 공통 Batch 인프라 Config (딱 1개)

```java
@Configuration
@EnableBatchProcessing
public class BatchInfrastructureConfig {
}
```

#### 역할

* JobRepository
* JobLauncher
* TransactionManager
* DataSource
* JobExplorer


### 3.2 도메인별 Job Config

```java
@Configuration
@RequiredArgsConstructor
public class UserBatchJobConfig {

    private final JobBuilderFactory jobBuilderFactory;

    @Bean
    public Job userSyncJob(Step userSyncStep) {
        return jobBuilderFactory.get("userSyncJob")
                .start(userSyncStep)
                .build();
    }
}
```

* Job 이름 충돌 방지
* 도메인 책임 명확
* 분리 배포 가능성 확보


### 3.3 도메인별 Step / Reader / Writer

```java
@Configuration
@RequiredArgsConstructor
public class UserStepConfig {

    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step userSyncStep(ItemReader<User> reader,
                             ItemWriter<User> writer) {
        return stepBuilderFactory.get("userSyncStep")
                .<User, User>chunk(1000)
                .reader(reader)
                .writer(writer)
                .build();
    }
}
```

## 4. 패키지 구조 예시

```text
batch
 ├─ infrastructure
 │   └─ BatchInfrastructureConfig
 ├─ user
 │   ├─ UserBatchJobConfig
 │   ├─ UserStepConfig
 │   ├─ UserItemReader
 │   └─ UserItemWriter
 ├─ order
 │   ├─ OrderBatchJobConfig
 │   └─ OrderStepConfig
```

### 5.1 내부 연결 구조

```text
JobRepository
  └─ JobBuilderFactory
      └─ JobBuilder
          └─ Job
```

* Job / Step은 반드시 JobRepository를 참조
* StepBuilderFactory 역시 TransactionManager를 내부적으로 사용


### 5.2 @EnableBatchProcessing 역할

`@EnableBatchProcessing`이 자동으로 등록하는 Bean들:

* JobRepository
* JobLauncher
* JobExplorer
* JobBuilderFactory
* StepBuilderFactory

#### 동작 규칙

* 이미 Bean이 있으면 → **사용**
* 없으면 → **Default 생성**


## 6. JobLauncher는 언제 쓰이는가?

### 직접 사용

* Scheduler
* REST API
* Event Listener

### 간접 사용

* `JobLauncherCommandLineRunner`
* `JobLauncherApplicationRunner`


## 7. InfrastructureConfig를 분리하는 이유

### 1) 책임 분리

| Config 종류      | 책임       |
| -------------- | -------- |
| Infrastructure | 배치 엔진 설정 |
| JobConfig      | 배치 유즈케이스 |
| StepConfig     | 처리 방식    |

