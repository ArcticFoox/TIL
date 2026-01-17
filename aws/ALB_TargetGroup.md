# AWS EC2 ALB 등록
현재 백엔드 EC2 만 https 연결된 ALB에 등록되어 있다.
크롤링 EC2도 같은 도메인 주소를 사용하기 위해 ALB에 등록을 필요로 했다.

### 대상 그룹(Target Group) 등록
ALB는 대상 그룹을 단위로 나뉘기에 새로운 대상 그룹 생성이 필요
대상 그룹에 등록을 위해선 `health check`를 위한 api가 필요
![image.png](/aws/img/tg1.png)
프로토콜과 포트는 application에 맞게 설정  
만약 nginx 같은 웹서버 사용 시 80포트로 설정

### ALB 규칙 등록
![image2.png](/aws/img/tg2.png)
![image3.png](/aws/img/tg3.png)
사용 중인 ALB에 path 별로 대상 그룹을 다르게 할 수 있기에,  
추가된 대상 그룹을 조건을 걸어 사용