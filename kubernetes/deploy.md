# k3s Kubernetes 배포
## k3s 설치
```bash
curl -sfL https://get.k3s.io | sh -
```
마스터 노드를 설치하고 안에서 worker 노드 만들어 사용하도록 구성
## Kubernetes 리소스 배포
```yaml
# -------------------
# 1. Database (PostgreSQL)
# -------------------
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  resources: { requests: { storage: 1Gi } }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  selector: { matchLabels: { app: postgres } }
  template:
    metadata: { labels: { app: postgres } }
    spec:
      containers:
        - name: postgres
          image: postgres:15  # 공식 이미지 사용
          env:
            - name: POSTGRES_DB
              value: "mydb"
            - name: POSTGRES_PASSWORD
              value: "password"
            - name: POSTGRES_USER
              value: "user"
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: pg-data
      volumes:
        - name: pg-data
          persistentVolumeClaim: { claimName: postgres-pvc }
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service  # 백엔드가 이 이름으로 찾음
spec:
  ports: [{ port: 5432 }]
  selector: { app: postgres }
---
# -------------------
# 2. Backend API
# -------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  selector: { matchLabels: { app: backend } }
  template:
    metadata: { labels: { app: backend } }
    spec:
      containers:
        - name: backend
          image: mybackimage
          imagePullPolicy: IfNotPresent
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "dev"
            - name: SPRING_DATASOURCE_URL
              value: "jdbc:postgresql://postgres-service:5432/mydb"
            - name: SPRING_DATASOURCE_USERNAME
              value: "user"
            - name: SPRING_DATASOURCE_PASSWORD
              value: "password"
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service 
spec:
  ports: [{ port: 8080 }]
  selector: { app: backend }
---
# -------------------
# 3. Frontend (BFF)
# -------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector: { matchLabels: { app: frontend } }
  template:
    metadata: { labels: { app: frontend } }
    spec:
      containers:
        - name: frontend
          image: myfrontimage
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8081
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "dev"
            - name: EXTERNAL_API_BASE-URL # 프론트 코드에 맞춰 변수명 수정
              value: "http://backend-service:8080/api/v1"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  ports: 
    - port: 8080
      targetPort: 8081
  selector: { app: frontend }
---
# -------------------
# 4. Ingress (외부 노출)
# -------------------
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port: { number: 8080 }
```

처음부터 보자면,  
pvc 를 설정해 postgres 데이터가 유지되도록 설정했고,  
postgres deployment 와 service 를 설정해 backend 가 접근할 수 있도록 구성  
backend deployment 에는 환경변수로 postgres service 이름을 넣어주어 연결  
frontend deployment 에는 backend service 이름을 넣어주어 연결  
마지막으로 ingress 를 설정해 외부에서 frontend service 로 접근할 수 있도록 구성  