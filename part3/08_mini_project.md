
# 🧑‍🏫 8주차 이론 강의 – Mini Project ①: 웹앱 Kubernetes 배포

## 1. 프로젝트 목표

- 지금까지 배운 개념(Pod, Deployment, Service, Ingress, ConfigMap, Secret, Volume, Helm)을 종합한다.
    
- 간단한 **웹 애플리케이션(예: Nginx, 간단한 HTML 서버)**을 Kubernetes에 배포한다.
    
- 외부 사용자가 브라우저로 접속 가능한 환경을 구축한다.
    

---

## 2. 프로젝트 구성 요소

### 2-1. Deployment

- 애플리케이션 컨테이너를 여러 개 실행
    
- 스케일링 및 롤링 업데이트 지원
    

### 2-2. Service

- Pod 앞단에서 고정 IP와 로드밸런싱 제공
    
- NodePort 또는 LoadBalancer를 활용해 외부 접속 지원
    
---
### 2-3. Ingress

- 도메인 기반 접근 가능
    
- `/` 경로로 라우팅 → 웹 애플리케이션 연결
    
- HTTPS 적용 가능 (선택)
    

### 2-4. ConfigMap & Secret

- ConfigMap: 애플리케이션 환경 설정 값 주입
    
- Secret: DB 접속 계정, 비밀번호 같은 민감정보 관리
    
---
### 2-5. Volume

- 애플리케이션의 정적 파일 또는 로그 데이터 저장
    

### 2-6. Helm Chart (선택)

- 전체 매니페스트를 Helm Chart로 패키징
    
- values.yaml로 환경별(dev/prod) 설정 가능
    

---

## 3. 아키텍처 흐름

1. 사용자가 브라우저에서 `http://myapp.local` 접속
    
2. 요청은 Ingress Controller → Ingress → Service로 전달
    
3. Service가 로드밸런싱을 통해 여러 Pod 중 하나로 전달
    
4. Pod 내부 컨테이너가 요청을 처리하고 응답 반환
    
5. ConfigMap/Secret으로 설정값 관리, Volume으로 데이터 유지
    

---

## 4. 프로젝트 예시 시나리오

- **웹앱**: 간단한 HTML 페이지 또는 Flask/Nginx 웹 서버
    
- **Deployment**: Pod 3개 실행
    
- **Service**: NodePort로 외부 접근 지원
    
- **Ingress**: `http://myapp.local` → Service 라우팅
    
- **ConfigMap**: “앱 색상=blue” 같은 환경변수 전달
    
- **Secret**: “DB 비밀번호” 저장 (실습에서는 단순 문자열)
    
- **Volume**: HTML 파일 공유
    

---

## 5. 학습 포인트

- 여러 Kubernetes 리소스를 조합하여 하나의 서비스를 완성하는 경험
    
- Pod, Service, Ingress의 연결 관계 실습
    
- ConfigMap/Secret을 실제 애플리케이션에 연동
    
- Volume을 이용해 데이터 영속성 개념 확인
    
- Helm으로 프로젝트 전체를 패키징하여 손쉽게 배포
    

---

## ✅ 정리

- 8주차는 단순 개념 학습이 아닌 **Mini Project** 형태로 진행
    
- 학생들이 직접 Kubernetes에 웹앱을 올리고 외부 브라우저에서 접속
    
- 이후 Part 2로 넘어가기 전, Kubernetes 기본 운영 흐름을 종합적으로 이해하는 단계
    

---

# 💻 8주차 실습 강의 – Mini Project ①: 웹앱 Kubernetes 배포

## 🎯 실습 목표

- 간단한 웹 애플리케이션(Nginx 기반)을 Kubernetes에 배포한다.
    
- Deployment, Service, Ingress, ConfigMap, Secret, Volume을 종합적으로 사용한다.
    
- 브라우저에서 `http://myapp.local`로 접속하여 웹앱을 확인한다.
    

---

## 1. 클러스터 준비

```bash
minikube start --driver=docker
kubectl get nodes
```

✅ `minikube Ready` 확인

---

## 2. ConfigMap과 Secret 생성

### 2-1. ConfigMap (`app-config.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: "blue"
  APP_MODE: "production"
```

```bash
kubectl apply -f app-config.yaml
kubectl get configmap app-config -o yaml
```

---
### 2-2. Secret (`app-secret.yaml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  db_user: YWRtaW4=       # base64(admin)
  db_password: cGFzc3dvcmQ= # base64(password)
```

```bash
kubectl apply -f app-secret.yaml
kubectl get secret app-secret -o yaml
```

---

## 3. Volume 준비

Pod 내부에서 공유할 임시 저장소(`emptyDir`) 사용

---

<!-- _class: small -->

## 4. Deployment 생성

### 4-1. Deployment (`myapp-deploy.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.25
        ports:
        - containerPort: 80
        env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: db_user
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: db_password
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        emptyDir: {}
```

---
```bash
kubectl apply -f myapp-deploy.yaml
kubectl get pods -l app=myapp
```

✅ Pod 3개가 Running 상태

---

## 5. Service 생성

### 5-1. Service (`myapp-svc.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f myapp-svc.yaml
kubectl get svc myapp-svc
```

---

<!-- _class: medium -->

## 6. Ingress 설정

### 6-1. Ingress Controller 활성화

```bash
minikube addons enable ingress
kubectl get pods -n kube-system
```

✅ `ingress-nginx-controller` Running 확인

### 6-2. Ingress 리소스 (`myapp-ingress.yaml`)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-svc
            port:
              number: 80
```

```bash
kubectl apply -f myapp-ingress.yaml
kubectl get ingress myapp-ingress
```
---
### 6-3. /etc/hosts 수정

로컬 PC에서 도메인을 `minikube` IP로 매핑

```bash
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts
```



## 7. 접속 테스트

브라우저에서 `http://myapp.local` 접속  
✅ Nginx 기본 페이지 표시

---

## 8. 확장 및 업데이트

- **스케일링**
    

```bash
kubectl scale deployment myapp-deploy --replicas=5
kubectl get pods -l app=myapp
```

- **이미지 업데이트**
    

```bash
kubectl set image deployment/myapp-deploy myapp=nginx:1.26
kubectl rollout status deployment/myapp-deploy
```

---

## 9. 정리

```bash
kubectl delete -f myapp-ingress.yaml
kubectl delete -f myapp-svc.yaml
kubectl delete -f myapp-deploy.yaml
kubectl delete -f app-config.yaml
kubectl delete -f app-secret.yaml
minikube stop
```

---

## ✅ 최종 결과

- ConfigMap, Secret, Volume을 포함한 Deployment를 구성
    
- Service와 Ingress로 외부에서 웹앱 접속 성공
    
- 스케일링 및 롤링 업데이트까지 경험
    

