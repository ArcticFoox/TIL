# ArgoCD
## 정의
ArgoCD는 Kubernetes 환경에서 애플리케이션의 지속적인 배포(CD)를 자동화하는 오픈 소스 도구  
GitOps 방식을 채택하여, Git 저장소를 단일 진실의 원천으로 사용하여 애플리케이션의 상태를 관리  
ArgoCD는 애플리케이션의 선언적 구성을 Git에 저장하고, 이를 Kubernetes 클러스터에 동기화하여 일관된 배포를 보장  

## 구성요소
주요 컴포넌트
argocd-server: ArgoCD의 API 서버로, 사용자 인터페이스와 CLI를 통해 상호작용
application-controller: Git 저장소와 Kubernetes 클러스터 간의 상태를 동기화하는 역할
repo-server: Git repo clone, Helm/Kustomize 렌더링 등을 담당
dex-server: OAuth2 및 OIDC 인증을 지원하는 인증 서버

## 핵심 리소스
Application: ArgoCD에서 관리하는 애플리케이션을 정의하는 리소스. Git 저장소, 대상 클러스터, 동기화 정책 등을 포함
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  source:
    repoURL: https://github.com/org/repo
    path: k8s/dev
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
```

## Sync & Drift 감지
ArgoCD는 Git 저장소와 Kubernetes 클러스터 간의 상태를 지속적으로 모니터링
애플리케이션의 상태가 Git과 일치하지 않을 경우 이를 감지하고, 자동 또는 수동으로 동기화할 수 있는 기능 제공
### 상태 유형
Synced: Git과 클러스터 상태가 일치  
OutOfSync: Git과 클러스터 상태가 불일치  
Missing / Extra: 클러스터에 존재하지 않거나 Git에 없는 리소스 감지  
누군가 kubectl edit 등으로 리소스 변경 시 ArgoCD UI에서 즉시 Drift 감지 -> Auto Sync 켜져 있으면 다시 원복
