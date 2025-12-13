# 자체 호스팅 실행기
- github hosting runner 에 비해 하드웨어, 운영 체제, 소프트웨어 도구를 더 많이 제어 가능

관리 계층 구조의 어디에서나 자체 호스팅 러너를 사용할 수 있음

# 내부 동작 과정
`./config.sh` 실행 시  
- GitHub에서 Runner 토큰 발급
- Runner가 Github에 등록 요청
- GitHub가 Runner에게 인증용 JWT, 설정값, 연결 주소 전달

Runner는 WebSocket 혹은 Service Bus 방식을 통해 github와 항상 연결 유지  

| 프로세스                | 역할                                   |
| ------------------- | ------------------------------------ |
| `Runner.Listener`   | GitHub와 연결 유지, Job 대기                |
| `Runner.Worker`     | Job을 실제 실행                           |
| `Runner.PluginHost` | Composite / JS Action 실행에 필요한 Plugin |

Listener가 GitHub Actions 서비스에 WebSocket 연결
Listener는 'Idle' 상태로 등록
GitHub에서 Job 할당 시 Listener가 Worker에게 작업 전달
Runner는 기본적으로 pull 모델이기에 Runner가 github에 계속 polling 요청을 보냄

### Job 실행 과정
Job 할당 시
GitHub 서버에서 Job definition(JSON) 다운
- 어떤 action steps?
- 어떤 환경변수?
- 어떤 secret?

필요한 Action 다운
`pre-job` 스크립트 실행
Job 단계(step) 순차 실행
각 단계마다 stdout/stderr를 GitHub 로그 스트리밍 엔드포인트로 Push

## Github Actions 서비스와의 통신 구조
### GitHub API
- Runner 등록
- Action 다운로드
- 체크아웃 정보 조작
### GitHub Actions Message Broker(WebSocket)
- Job 할당 이벤트 수신
- 상태 PING
### GitHub Actions Log/Artifacts Server
- Log chunk push
- Artifact 업로드
통신은 모두 HTTPS + 인증된 JWT 기반

# 요약
<b>Self-hosted runner = GitHub와 WebSocket을 유지하며 Job을 pull/poll로 받아 실행하는 Agent 프로그램
설치 시 GitHub에 Runner 등록</b>  
- Listener 프로세스가 GitHub Actions와 상시 연결
- Job이 생기면 Worker 프로세스가 실행
- 로그는 실시간으로 GitHub로 스트리밍
- Job 완료 후 상태 리포트
- 다시 idle 상태로 돌아감