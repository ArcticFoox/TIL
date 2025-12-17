# spring batch 6.0 버전
## 변경사항
### 의존성 업그레이드
- Spring Framework 7.x
- Spring Integration 7.x
- Spring Data 4.x
- Spring AMQP 4.x
- Spring for Apache Kafka 4.x
- Micrometer 1.16
  
관련 라이브러리 전체를 함께 맞춰야 안정적으로 동작

## 배치 인프라 설정 방식 변화
### 공통 속성과 저장소 속성 분리
기존에는 배치 인프라 설정이 JDBC 기반으로 강하게 묶여 있었지만 이젠 저장소별 분리
- JDBC 저장소 설정: @EnableJdbcJobRepository
- MongoDB 저장소 설정: @EnableMongoJobRepository

### 기본 배치 설정의 변화
- 기본값으로 데이터베이스가 필요없는 "리소스리스 배치 인프라" 제공
- JobExplorer와 JobLauncher가 더 이상 필요 없음, JobRepository와 JobOperator로 통합됨
- JobRegistrySmartIntializingSingleton도 제거되어 설정 간소화

## Deprecated 및 제거된 API
TansactionManager, JobRepositoryFactoryBean 등 일부 API가 제거되거나 대체됨