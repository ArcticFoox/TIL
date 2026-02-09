# k8s 에 대한 정리

docker 사용 시

- 컨테이너가 죽으면? → **수동으로 다시 실행**
- 트래픽이 많아지면? → **컨테이너 수동 복제**
- 서버가 죽으면? → **서비스 전체 장애**
- 버전 업데이트? → **중단 시간 발생**

### k8s 역할

컨테이너를 자동으로 관리하는 운영 시스템

- 컨테이너 자동 실행 / 재시작
- 트래픽에 따라 자동 확장 (Scale)
- 무중단 배포 (Rolling Update)
- 서버 장애 시 자동 복구
- 네트워크 & 로드밸런싱 제공

```java
[ User ]
   |
 kubectl
   |
[ Control Plane ]
  ├─ API Server
  ├─ Scheduler
  ├─ Controller Manager
  └─ etcd
        |
[ Worker Node ]
  ├─ kubelet
  ├─ kube-proxy
  └─ Containers (Pods)

```

## Cluster 구성 요소

### cluster

kubernetes가 관리하는 전체 환경

여러 대의 Node 묶음

### control Plane

kube-apiserver : 모든 요청의 입구

etcd : 클러스터 상태 저장소

scheduler : 어떤 노드에 Pod 배치할지 결정

controller-manager : 상태 유지 (pod가 죽으면 다시 띄우는 역할)

### worker Node

kubelet

kube-proxy

containers(Pods)

### Pod

k8s의 최소 실행단위

컨테이너 1개 이상을 묶은 것(정말로 함께 해야하는 것이 아니라면 1개의 container 만 존재하게 하는 게 정석)

### Deployment

Pod를 원하는 개수만큼 유지해주는 관리자

Pod는 Deployment로 관리

### Service

Pod는 IP가 계속 변경됨

Service를 통해 고정된 접근 지점 마련

### Ingress / Gateway API

HTTP/HTTPS 진입점

도메인, TLS, Path 기반 라우팅 제공

```java
https://api.example.com
   |
Ingress
   |
Service
   |
Pods
```

### Pod down 시

Deployment 감지

새 Pod 생성

Service에서 IP 감지

### 배포 흐름

```java
1. 코드 작성
2. Docker Image 생성
3. Image Registry push
4. Deployment 적용
5. Pod 생성
6. Service / Ingress 연결
```