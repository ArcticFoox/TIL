# Skaffold
빌드, 푸시 및 배포하기 위한 워크플로 처리 CI/CD 파이프라인을 만들기 위한 빌딩 블록 제공
## 특징
- 다중 환경 지원: 개발, 스테이징, 프로덕션 등 다양한 환경에 맞게 설정 가능
- 빠른 피드백 루프: 코드 변경 시 자동으로 빌드 및 배포이 이루어져 개발 속도 향상
- 다양한 빌드 및 배포 도구 통합: Docker, Kaniko, Helm, Kustomize 등과 호환
- 로컬 및 원격 클러스터 지원: Minikube, Kind, GKE, EKS 등 다양한 Kubernetes 클러스터와 연동 가능
- 확장성: 플러그인 시스템을 통해 사용자 정의 빌드 및 배포 프로세스 추가 가능
## 주요 구성 요소
- skaffold.yaml: Skaffold의 설정 파일로, 빌드, 푸시 및 배포 단계를 정의
- 빌드(Build): 애플리케이션 이미지를 빌드하는 단계
- 푸시(Push): 빌드된 이미지를 컨테이너 레지스트리에 푸시하는 단계
- 배포(Deploy): Kubernetes 클러스터에 애플리케이션을 배포하는 단계
## 예시
```yaml
apiVersion: skaffold/v2beta26
kind: Config
metadata:
  name: my-app
build:
  artifacts:
    - image: my-app-image
      context: .
      docker:
        dockerfile: Dockerfile
  local:
    push: false
deploy:
  kubectl:
    manifests:
      - k8s/deployment.yaml
      - k8s/service.yaml
```
## 사용법
1. Skaffold 설치: 공식 문서에 따라 Skaffold를 설치
2. skaffold.yaml 작성: 프로젝트 루트에 skaffold.yaml 파일 생성 및 설정
3. 개발 모드 실행: `skaffold dev` 명령어로 개발 모드 시작
4. 빌드 및 배포: `skaffold build` 및 `skaffold deploy` 명령어로 수동 빌드 및 배포 수행
5. CI/CD 통합: Jenkins, GitLab CI 등과 연동하여 자동화된 파이프라인 구축
## 참고 자료
- [Skaffold 공식 문서](https://skaffold.dev/docs/)