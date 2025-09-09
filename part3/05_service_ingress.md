# 🧑‍🏫 5주차 이론 강의 – Service와 Ingress

## 1. Pod 네트워크 한계

- Pod는 생성될 때마다 IP가 새로 할당됨
    
- Pod가 죽고 새로 만들어지면 IP가 바뀜
    
- 외부에서 Pod에 직접 접근 불가능  
    👉 **안정적인 접근 지점**이 필요 → Service
    

---

## 2. Service 개념

- Pod 집합에 고정된 네트워크 엔드포인트를 제공
    
- **로드밸런싱**: 여러 Pod 중 하나로 트래픽 분산
    
- **서비스 디스커버리**: Pod IP 대신 서비스 이름으로 접근
    

---



## 3. Service 유형

### 3-1. ClusterIP (기본값)

- 클러스터 내부에서만 접근 가능
    
- 내부 마이크로서비스 통신에 사용
    

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---
### 3-2. NodePort

- 클러스터 외부에서 접근 가능
    
- 각 노드의 특정 포트(NodePort)를 열어줌
    
- URL 예: `http://<NodeIP>:30080`
    

### 3-3. LoadBalancer

- 클라우드 환경에서 외부 로드밸런서를 생성
    
- AWS, GCP, Azure에서 사용
    

---

## 4. Ingress 개념

- 여러 서비스에 대한 **외부 진입점(HTTP/HTTPS 라우팅)** 제공
    
- 도메인/URL 경로 기반 라우팅 지원
    
- TLS(HTTPS) 인증서 적용 가능
    
- **Ingress Controller** 필요 (예: NGINX Ingress Controller)
    

---

## 5. Ingress 동작 구조

1. 사용자가 `http://example.com/foo` 요청
    
2. Ingress Controller(Nginx)가 요청 수신
    
3. Ingress 규칙에 따라 `/foo` 요청을 특정 Service로 전달
    
4. Service가 Pod로 요청 전달
    

---

## 6. Ingress 매니페스트 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

---

## 7. Service vs Ingress 비교

- Service: Pod 접근을 위한 **네트워크 엔드포인트**
    
- Ingress: 여러 서비스 앞단에서 **HTTP 라우팅 및 도메인 관리**
    

---

## 8. 실제 활용 예시

- **ClusterIP**: 내부 DB 서비스(MySQL) 접근
    
- **NodePort**: 개발·테스트 환경에서 외부 접근
    
- **LoadBalancer**: 클라우드 환경에서 외부 사용자 접근
    
- **Ingress**: 실제 운영환경에서 웹 서비스 라우팅 + HTTPS 적용
    

---

## ✅ 정리

- Service는 Pod 접근을 위한 안정적인 엔드포인트 제공
    
- Service 타입: ClusterIP(내부), NodePort(외부 포트), LoadBalancer(클라우드)
    
- Ingress는 외부 트래픽을 여러 서비스로 라우팅하고 HTTPS를 지원
    
- 이후 실습에서 Service와 Ingress를 직접 생성하고 테스트함
    

---

# 💻 5주차 실습 강의 – Service와 Ingress

## 🎯 실습 목표

- Pod와 Deployment를 배포한 뒤 Service로 접근 지점을 만든다.
    
- ClusterIP, NodePort, LoadBalancer의 차이를 확인한다.
    
- Ingress Controller를 설치하고, Ingress를 통해 도메인 기반 라우팅을 실습한다.
    

---
<!-- _class: medium -->

## 1. 준비

### 1-1. 클러스터 실행

```bash
minikube start --driver=docker
```

### 1-2. 기본 Deployment 생성 (`nginx-deploy.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---
```bash
kubectl apply -f nginx-deploy.yaml
kubectl get pods -l app=nginx
```

✅ Pod 2개가 Running 상태 확인

---


<!-- _class: medium -->

## 2. ClusterIP Service

### 2-1. 매니페스트 (`nginx-svc-clusterip.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-clusterip
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### 2-2. 적용

```bash
kubectl apply -f nginx-svc-clusterip.yaml
kubectl get svc
```

✅ `CLUSTER-IP`가 할당된 Service 확인

---
### 2-3. 내부 접속 확인

```bash
kubectl run curl --image=radial/busyboxplus:curl -i --tty --rm
```

Pod 내부에서:

```bash
curl http://nginx-svc-clusterip:80
```

✅ nginx 기본 페이지 HTML 출력

---

<!-- _class: medium -->

## 3. NodePort Service

### 3-1. 매니페스트 (`nginx-svc-nodeport.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-nodeport
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

### 3-2. 적용

```bash
kubectl apply -f nginx-svc-nodeport.yaml
kubectl get svc
```

✅ `PORT(S)`에 `80:30080/TCP` 표시 확인

### 3-3. 외부 접속 확인

```bash
minikube service nginx-svc-nodeport --url
```

출력된 URL 접속 → 브라우저에서 nginx 환영 페이지 확인

---

<!-- _class: medium -->

## 4. LoadBalancer Service (Minikube 환경)

### 4-1. 매니페스트 (`nginx-svc-lb.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-lb
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

### 4-2. 적용

```bash
kubectl apply -f nginx-svc-lb.yaml
minikube service nginx-svc-lb --url
```

✅ URL 접속해 nginx 페이지 확인

---

## 5. Ingress Controller 설치

### 5-1. NGINX Ingress Controller 설치

```bash
minikube addons enable ingress
kubectl get pods -n kube-system
```

✅ `nginx-ingress-controller` Pod Running 확인

---

<!-- _class: medium -->

## 6. Ingress 리소스 생성

### 6-1. Ingress 매니페스트 (`nginx-ingress.yaml`)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc-clusterip
            port:
              number: 80
```

### 6-2. 적용

```bash
kubectl apply -f nginx-ingress.yaml
kubectl get ingress
```
---
### 6-3. 로컬 호스트 파일 수정 (Linux/Mac)

```bash
sudo -- sh -c "echo '127.0.0.1 myapp.local' >> /etc/hosts"
```

### 6-4. 접속 확인

브라우저에서 `http://myapp.local` → nginx 페이지 표시

---

## 7. 정리

```bash
kubectl delete -f nginx-ingress.yaml
kubectl delete -f nginx-svc-lb.yaml
kubectl delete -f nginx-svc-nodeport.yaml
kubectl delete -f nginx-svc-clusterip.yaml
kubectl delete -f nginx-deploy.yaml
minikube stop
```

---

## ✅ 최종 결과

- ClusterIP로 내부 통신, NodePort/LoadBalancer로 외부 통신 확인
    
- Ingress Controller 설치 및 도메인 기반 라우팅 체험
    
- Pod → Service → Ingress 네트워크 구조 완전히 이해
    

