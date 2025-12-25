# Quartz
## 장점
- DB에 스케줄 정보를 저장하여 서버가 재시갇외어도 유지 가능
- 클러스터링을 지원하여 여러 노드에서 동일한 작업을 수행 가능
- 작업을 동적으로 추가/삭제/수정할 수 있음
- 다양한 트리거 옵션을 지원하여 세밀한 스케줄링 가능

## 단점
- 여러 설장 파일과 객체를 다루어야함
- trigger 및 job 설정을 정교하게 다루어야 함
- 내부적으로 데이터베이스와 연동하여 상태를 관리하기에, 대규모 trigger나 작업 스케줄링 경우 자원 소모
- 클러스터링과 같은 기능을 활용할 때는 설정 및 관리가 까다로움

## 용어
- Scheduler : 전체 Quartz 스케줄링 시스템을 의미하며, Job과 Trigger를 관리
- Job : 실행할 작업 (비즈니스 로직)
- JobDetail : Job의 메타데이터 (Job 클래스, 이름, 그룹 등)
- Trigger :  Job이 실행될 시점을 결정하는 스케줄러 (예: CronTrigger, SimpleTrigger)
- JobStore :  Job과 Trigger 정보를 저장하는 저장소 (RAMJobStore, JDBC JobStore)
- RAMJobStore :  메모리에 Job을 저장하는 방식 (애플리케이션 종료 시 데이터 소멸)
- JDBC JobStore :  Job을 DB에 저장하여 애플리케이션 종료 후에도 유지 가능
- Scheduler Instance :  Quartz 클러스터에서 실행 중인 개별 Quartz 인스턴스
- Misfire :  Job이 예약된 시간에 실행되지 못했을 때 발생하는 상태
- Listener :  특정 이벤트 (Job 실행, 완료 등)에 대한 처리를 담당하는 클래스

