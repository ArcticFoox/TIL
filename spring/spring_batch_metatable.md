# Spring Batch MetaTable
## 구성요소
### BATCH_JOB_INSTANCE
- JOB_INSTANCE_ID: PK
- JOB_NAME: Job 이름

Job Parameter에 따라 생성되는 테이블
Spring Batch가 실행될 때 외부에서 받을 수 있는 파라미터
예시) 특정 날짜를 Job Parameter로 넘기면 Spring Batch에서는 해당 날짜 데이터로 조회/가공/입력 등의 작업
  
같은 Batch Job이라도 Job Parameter가 다르면 다른 JOB_INSTANCE_ID로 생성
  
### BATCH_JOB_EXECUTION
JOB_EXECUTION와 JOB_INSTANCE는 부모-자식 관계
JOB_EXECUTION은 자신의 부모 JOB_INSTANCE가 성공/실패했던 모든 내역을 가지고 있음
동일한 Job Parameter로 2번 실행했는데 같은 파라미터로 실행되었다는 에러가 발생하지 않으면, 그건 성공한 기록이 없기 때문
동일한 Job Parameter로 성공한 기록이 있을 때만 재수행이 되지 않음
