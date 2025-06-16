# AWS Application Load Balancer

ALB는 주로 HTTP/HTTPS 트래픽을 처리하는 데 사용

애플리케이션 계층에서의 로드 밸런싱을 제공

HTTP/HTTPS 요청을 기반으로 트래픽을 분산시키기 때문

ALB는 URL 경로 기반 라우팅 지원

-특정 URL 경로에 따라 트래픽을 다른 대상 그룹으로 라우팅

호스트 기반 라우팅 지원

-특정 호스트 이름에 따라 트래픽을 다른 대상 그룹으로 라우팅

WebSocket 및 HTTP/2를 지원

-실시간 통신 및 효율적인 데이터 전송 가능

Docker 컨테이너화된 애플리케이션과의 통합을 용이

고정 IP 주소(Elastic IP)를 사용 불가

Sticky Session 사용 가능

## Host(호스트) and Path(경로) Based Routing 기능

### 호스트 기반

클라이언트가 요청한 접속 URL의 FQDN(완전 도메인 이름)에 따라 라우팅 할 수 있는 기능

ex) "http://www1.example.com"일 경우 웹 서버 1에 접속, "http://www2.example.com"일 경우 웹 서버 2에 접속

### 경로 기반

클라이언트가 요청한 접속 URL의 경로에 따라 라우팅 할 수 있는 기능

ex) "http://www.example.com/web1/"일 경우 웹 서버 1에 접속, "http://www.example.com/web2/"일 경우 웹 서버 2에 접속

### URL Query 문자열 기반

클라이언트가 요청한 접속 URL의 쿼리 문자열에 따라 라우팅할 수 있는 기능

ex) "http://www.example.com/web?lang=kr"일 경우 한국어 사이트로 접속, "http://www.example.com/web?lang=en"일 경우 영어 사이트로 접속

## Sticky Session 기능

세션 정보가 저정된 쪽으로 지속적 연결

Session Cookie의 유효시간 동안 클라이언트가 동일한 백엔드 인스턴스(서버)에 접근할 수 있도록 하는 기능

### 작동방식

- ALB는 첫 번째 요청을 처리할 때, 클라이언트에게 **쿠키**를 발급합니다.
- 이 쿠키에는 요청을 처리한 **Target Group의 특정 대상(Target)** 정보가 포함되어 있습니다.
- 이후 요청에서 클라이언트가 해당 쿠키를 포함하면, ALB는 동일한 대상에 요청을 라우팅합니다.
- 이로 인해 **세션 정보(예: 로그인 상태)** 가 서버에 저장되어 있어야 하는 경우, 같은 서버로 계속 요청이 가므로 상태 유지가 가능합니다.

### 설정방법

콘솔

1. **Target Group** 생성 또는 수정 시
2. **Attributes** > **Stickiness** 옵션을 활성화
3. **Stickiness type**: `app_cookie` 선택
4. **Cookie name**: `AWSALB` 또는 커스텀
5. **Duration**: 지속 시간(초 단위, 예: 300초 = 5분)

### 쿠키 종류

AWSALB: 기본 애플리케이션 쿠키 이름

AWSALBTG: 여러 타겟 그룹을 사용할 때 각 그룹마다 고유 쿠키 발급

### 주의 사항

- Sticky session은 **로드 밸런싱의 이점을 일부 희생**합니다. 하나의 인스턴스에 트래픽이 몰릴 수 있기 때문입니다.
- 클라이언트가 쿠키를 차단하거나 삭제하면 세션이 끊깁니다.
- Target이 비정상 상태(헬스 체크 실패 등)가 되면 세션이 강제로 다른 대상으로 이동됩니다.
- 상태 저장 방식은 **수평 확장성에 한계**를 줄 수 있으므로, 세션 저장소(Redis 등)를 외부에 두는 방식이 더 유연합니다.

## Listener Rules

수신한 요청을 어떻게 처리할지에 대한 규칙을 설정한 기능

호스트/경로 기반 라우팅은 Listener Rules을 통해 라우팅