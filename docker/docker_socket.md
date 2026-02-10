# Docker Socket
## Socket
소켓 파일은 컴퓨터 내에서 프로세스 간 통신을 위한 인터페이스를 제공하는 파일 형태의 통신 채널
주로 리눅스 환경에서 `*.sock` 확장자로 나타나며, 로컬에서 프로세스 간 네트워크 통신 없이도 데이터를 주고받을 수 있도록 함

## Docker Socket
- Docker 소켓(`/var/run/docker.sock`)은 Docker 데몬과 통신하기 위한 인터페이스 역할을 하는 유닉스 소켓 파일  
- 주요역할
  - 사용자가 입력한 명령어를 클라이언트에서 Docker 데몬으로 전달
  - 데몬이 전달받은 명령어를 수행
  - 데몬이 수행한 결과를 다시 클라이언트로 반환
- Docker 소켓을 통해 Docker CLI와 Docker 데몬 간의 통신이 이루어짐

## socket file mount
- socket file을 container에 마운트하면, 해당 컨테이너가 호스트의 Docker 데몬과 직접 통신할 수 있게 됨
- 이를 통해 컨테이너 내부에서 Docker 명령어를 실행하여 호스트의 Docker 리소스를 관리할 수 있음  

예시
```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock -it docker
```
- 위 명령어는 호스트의 Docker 소켓 파일을 컨테이너 내부에 마운트하여,  
컨테이너가 호스트의 Docker 데몬과 통신할 수 있도록 설정

## 통신 과정
1. Docker 데몬이 실행되면 `/var/run/docker.sock` 파일이 생성
   - 소켓 파일은 Unix 소켓을 통해 클라이언트와 데몬 간 통신을 담당하는 엔드포인트 역할 수행
2. 사용자가 Docker CLI를 통해 명령어를 입력하면, Docker 클라이언트는 이 명령을 HTTP 요청으로 변환하여 소켓 파일을 통해 데몬에 전달
   - Docker는 HTTP 기반 REST API를 사용하여 명령을 처리하며, 이 요청이 TCP 대신 로컬 Unix 소켓 파일을 통해 전달
     - HTTP 통신 방식은 클라이언트가 HTTP 요청을 보내고, 서버가 HTTP 응답을 반환하는 패턴
     - HTTP 프로토콜 형식을 따르는 것이지 실제 네트워크를 통한 HTTP 통신(TCP/IP)은 아님
     - Unix 소켓은 네트워크 없이 로컬에서 클라이언트와 데몬 간에 데이터 전송을 처리
3. Docker 데몬은 클라이언트로부터 받은 HTTP 요청을 처리
4. 데몬이 요청을 처리한 후, 그 결과를 다시 HTTP 응답으로 변환하여 소켓 파일을 통해 클라이언트에 반환

출처  
[velog](https://velog.io/@reasonoflife39/Docker%EC%9D%98-%EC%86%8C%EC%BC%93-%ED%8C%8C%EC%9D%BC-%EB%8D%B0%EB%AA%AC%EA%B3%BC-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%ED%86%B5%EC%8B%A0)