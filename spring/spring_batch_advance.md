# Spring Batch
## 필요 이유
단순 스크립트로는 해결이 어려운 대량 처리 안정성
- 중단 시점부터의 재시작
- 실패 이력 관리
트랜잭션과 병렬 처리
- 일정 단위(chunk)로 나눠 commit
- Thread / Partitioning / Remote Chunking 기반의 병렬 처리 지원
운영/모니터링 필수 요소 제공
- Job/Step 실행 이력 저장
- 성공/실패 여부 기록
- 재시도/재처리 가능
Spring 기반이라 DI, AOP 등 활용 가능
  
## 구성 요소
### Job
배치 작업 전체 단위
여러 Step으로 구성
### Step
Job을 구성하는 단위 작업
Chunk 기반 또는 Tasklet 기반 구성
### ItemReader
데이터 읽기
### ItemProcessor
데이터 가공, 검증, 필터링
### ItemWriter
데이터 저장
### JobRepository
Job/Step 실행 이력 저장소
실패 시 재시작에 사용

## 처리 방식
Chunk-Oriented Processing
- 데이터를 일정 단위(chunk)로 읽고 처리 후 저장
- 트랜잭션 단위로 commit/rollback 가능
- 예: 100건 단위로 읽고 처리 후 DB에 저장

1. Reader가 데이터를 하나씩 읽음
2. Processor가 데이터를 가공/검증
3. Writer가 일정 단위(chunk)로 데이터를 저장
4. 이 단위를 반복

## 상세 요소
### Job
```java
@Bean
public Job exampleJob(JobRepository jobRepository, Step step) {
    return new JobBuilder("exampleJob", jobRepository)
            .start(step)
            .build();
}
```
실행 이력은 JobRepository가 자동 관리
### Step
- Chunk 기반 Step
```java
@Bean
public Step exampleChunkStep(JobRepository jobRepository, 
                             PlatformTransactionManager transactionManager,
                             ItemReader reader,
                             ItemProcessor processor,
                             ItemWriter writer) {

    return new StepBuilder("exampleChunkStep", jobRepository)
            .<InputType, OutputType>chunk(100, transactionManager)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
}
```
- Tasklet 기반 Step
```java
@Bean
public Step simpleTaskletStep(JobRepository jobRepository, TransactionManager tm) {
    return new StepBuilder("simpleStep", jobRepository)
            .tasklet((contribution, chunkContext) -> {
                System.out.println("단순 작업 실행");
                return RepeatStatus.FINISHED;
            }, tm)
            .build();
}
```
  
### Reader/Processor/Writer
<b> Reader </b>
JdbcPagingItemReader (DB를 페이지 단위로 조회)
JdbcCursorItemReader (DB 커서 기반 → 대량 처리 시 부하)
JpaPagingItemReader
FlatFileItemReader (CSV, TXT)
XmlItemReader
KafkaItemReader
Rest API 기반 Custom Reader
<b>Processor</b>
데이터 가공, 검증, 필터링 로직 구현
null return 시 필터링
<b> Writer </b>
JdbcBatchItemWriter (DB batch insert/update)
JpaItemWriter
FlatFileItemWriter (CSV 파일 저장)
Custom API Writer
  
## 실행 방식
### Boot에서 자동 실행
`spring.batch.job.enabled=true` 서버 실행 시 모든 Job 자동 실행
### JobLauncher로 수동 실행
```java
@Autowired JobLauncher jobLauncher;
@Autowired Job job;

public void runJob() {
    jobLauncher.run(job, new JobParametersBuilder()
           .addLong("time", System.currentTimeMillis())
           .toJobParameters());
}
```
### 스케줄링과 연동
```java
@Scheduled(cron = "0 0 3 * * *")
public void scheduleJob() {
    runJob();
}
```
  
## JobRepository 기반 실행 이력 관리
DB 테이블 자동 생성
- BATCH_JOB_INSTANCE
- BATCH_JOB_EXECUTION
- BATCH_STEP_EXECUTION
- BATCH_JOB_EXECUTION_PARAMS
- ETC.
이를 기반으로 재시작/중단/실패 포인트 관리
  
## 병렬 처리
### Multi-Threaded Step
스레드 풀로 병렬 처리
### Partitioning
데이터를 파티션으로 나눠 병렬 처리
### Remote Chunking
Reader/Processor는 마스터에서, Writer는 워커에서 실행
마스터-워커 간 메시지 큐로 통신
### Spring Cloud Task + Spring Cloud Data Flow
컨테이너 기반 배치 분산 처리

