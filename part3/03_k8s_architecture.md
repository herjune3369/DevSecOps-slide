
# 🧑‍🏫 3주차 이론 강의 – Kubernetes 아키텍처

## 1. Kubernetes란?

- Google이 내부 시스템 **Borg** 경험을 바탕으로 개발한 오픈소스 컨테이너 오케스트레이션 플랫폼
    
- 수십~수천 개 컨테이너를 자동으로 관리
    
- 주요 기능
    
    - **배포 자동화**: 원하는 상태(Desired State)를 선언하면 자동 반영
        
    - **확장성**: 트래픽 변화에 따라 Pod 개수 자동 조정
        
    - **복원력**: 장애 발생 시 자동 복구(Self-Healing)
        
    - **서비스 관리**: 네트워크 라우팅과 로드밸런싱 내장
        

---

## 2. 핵심 개념

- **클러스터(Cluster)**
    
    - Control Plane(제어 영역) + Worker Node(실행 영역)의 집합
        
- **노드(Node)**
    
    - 컨테이너가 실제로 실행되는 물리/가상 머신
        
- **Pod**
    
    - 쿠버네티스에서 배포되는 가장 작은 실행 단위
        
    - 하나 이상의 컨테이너 + 스토리지 + 네트워크 정보 포함


---
- **ReplicaSet**
    
    - 동일한 Pod 개수를 유지하는 컨트롤러
        
- **Deployment**
    
    - Pod/ReplicaSet을 선언적으로 관리, 롤링 업데이트·롤백 지원
        
- **Service**
    
    - Pod의 고정 네트워크 엔드포인트 제공, 로드밸런싱 담당
        

---

<!-- _class: medium -->

## 3. 아키텍처 구성

### Control Plane (클러스터 두뇌)

- **API Server**
    
    - 모든 요청의 진입점, kubectl/CLI/다른 서비스와 통신
        
- **etcd**
    
    - 클러스터의 모든 상태를 저장하는 분산 키-값 DB
        
- **Scheduler**
    
    - 새로 생성된 Pod를 실행할 적절한 노드 선택
        
- **Controller Manager**
    
    - 클러스터 상태를 지속적으로 감시, 원하는 상태와 실제 상태를 일치시킴
        
- **Cloud Controller Manager** (선택적)
    
    - 클라우드 인프라(AWS, GCP 등)와 연동
        
---

### Worker Node (작업 실행 단위)

- **kubelet**
    
    - 노드 내 Pod 실행 관리, API Server와 통신
        
- **kube-proxy**
    
    - 네트워크 트래픽을 적절한 Pod로 전달, 로드밸런싱 수행
        
- **Container Runtime**
    
    - 컨테이너 실행 담당 (Docker, containerd, CRI-O 등 지원)
        

---

## 4. Kubernetes 동작 흐름

1. 사용자가 `kubectl`로 Deployment 생성 요청
    
2. **API Server**가 요청 수신 후 **etcd**에 상태 저장
    
3. **Scheduler**가 Pod를 어떤 노드에 배치할지 결정
    
4. 해당 노드의 **kubelet**이 컨테이너 실행
    
5. **kube-proxy**가 서비스 요청을 Pod로 라우팅
    
6. Controller Manager가 상태를 감시, 필요한 경우 자동 조치
    

---

## 5. Kubernetes의 장점

- **Self-healing**: Pod 장애 발생 시 자동 재생성
    
- **Auto-scaling**: HPA(Horizontal Pod Autoscaler)로 부하에 따라 Pod 개수 조절
    
- **Service Discovery & Load Balancing**: 서비스 이름 기반으로 Pod 자동 연결
    
- **리소스 최적화**: CPU/메모리 자원을 효율적으로 분배
    
- **플랫폼 독립성**: 온프레미스, 퍼블릭 클라우드 어디서든 동일한 방식 운영
    
- **선언형 관리**: YAML/JSON으로 “원하는 상태” 정의 → 자동 적용
    

---

## 6. 실제 활용 사례

- **Google, Spotify, 쿠팡**: 수천 개 마이크로서비스 운영
    
- **금융기관**: 보안성과 고가용성이 필요한 서비스 배포
    
- **스타트업**: 빠른 배포와 확장을 위한 표준 선택
    

---

## ✅ 정리

- Kubernetes는 컨테이너 기반 서비스 운영의 사실상 표준 플랫폼
    
- Control Plane은 관리와 제어, Worker Node는 실행 담당
    
- Pod, ReplicaSet, Deployment, Service가 가장 핵심적인 리소스
    
- 이후 실습(4주차)부터는 Pod와 Deployment를 직접 다루게 됨
    


---

# 💻 3주차 실습 강의 – Kubernetes 아키텍처 체험

## 🎯 실습 목표

- Minikube를 사용해 Kubernetes 클러스터를 실행한다.
    
- `kubectl` 기본 명령어를 익힌다.
    
- Pod와 Deployment를 생성하고 동작을 확인한다.
    
- Service를 통해 외부에서 Pod에 접속한다.
    

---

## 1. 실습 환경 준비

### 1-1. 필수 설치

- **Docker** (컨테이너 실행 환경)
    
- **kubectl** (Kubernetes CLI)
    
- **Minikube** (로컬 Kubernetes 클러스터 실행 툴)
    

### 1-2. 설치 확인

```bash
docker --version
kubectl version --client
minikube version
```

✅ 각 명령에서 버전이 출력되면 정상

---
<!-- _class: medium -->

## 2. Minikube 클러스터 실행

1. Minikube 시작
    

```bash
minikube start --driver=docker
```

2. 클러스터 상태 확인
    

```bash
minikube status
```

✅ 출력 예시:

```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

3. 노드 확인
    

```bash
kubectl get nodes
```

✅ `minikube Ready` 확인

---

<!-- _class: medium -->

## 3. Pod 직접 생성

1. Pod YAML 작성 (`nginx-pod.yaml`)
    

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

2. Pod 생성
    

```bash
kubectl apply -f nginx-pod.yaml
```

3. Pod 확인
    

```bash
kubectl get pods
kubectl describe pod nginx-pod
```

✅ STATUS가 `Running`이면 성공

---
<!-- _class: medium -->

## 4. Deployment 생성

1. Deployment YAML 작성 (`nginx-deploy.yaml`)
    

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

2. Deployment 생성
    

```bash
kubectl apply -f nginx-deploy.yaml
```

3. 확인
    

```bash
kubectl get deployments
kubectl get pods -l app=nginx
```

✅ Pod가 3개 실행 중인 것 확인

---

<!-- _class: medium -->

## 5. Service 생성

1. Service 생성 (`nginx-svc.yaml`)
    

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```

2. 적용
    

```bash
kubectl apply -f nginx-svc.yaml
```

3. 서비스 확인
    

```bash
kubectl get svc
```

✅ 출력 예시:

```
nginx-service   NodePort   10.102.89.12   <none>   80:30080/TCP   1m
```

---

## 6. 외부 접속 테스트

1. 서비스 URL 확인
    

```bash
minikube service nginx-service --url
```

✅ 출력 예시:

```
http://127.0.0.1:32744
```

2. 브라우저에서 접속 → **nginx 환영 페이지** 확인
    

---

## 7. 리소스 정리

```bash
kubectl delete -f nginx-svc.yaml
kubectl delete -f nginx-deploy.yaml
kubectl delete -f nginx-pod.yaml
minikube stop
```

---

## ✅ 최종 결과

- Pod와 Deployment의 차이를 이해하고,
    
- Service를 통해 외부에서 Pod에 접근하는 경험 완료
    
- Kubernetes 아키텍처(Controller, Scheduler, kubelet 등)가 어떻게 동작하는지 실제 확인
    

