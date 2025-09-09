
# 🧑‍🏫 4주차 이론 강의 – Pod, ReplicaSet, Deployment

## 1. Pod

- 쿠버네티스에서 배포되는 가장 작은 실행 단위
    
- 1개 이상의 컨테이너 + 공유 네트워크(IP) + 공유 스토리지(Volume)
    
- 컨테이너가 죽으면 Pod 전체가 재시작됨
    
- 보통은 컨테이너 1개 = Pod 1개 구조로 사용
    

---

## 2. ReplicaSet

- 특정 수의 Pod를 항상 유지하는 리소스
    
- 예: nginx Pod 3개 유지 선언 → 하나 죽으면 새로 생성
    
- Pod 개수를 수동으로 조절할 필요 없이 자동 관리
    
- 단독으로 잘 사용하지 않고 Deployment에 포함되어 사용됨
    

---

## 3. Deployment

- Pod와 ReplicaSet을 **선언형(Declarative)** 으로 관리
    
- 주요 기능
    
    - 원하는 Pod 개수 유지
        
    - 롤링 업데이트: 점진적으로 새 버전 배포
        
    - 롤백: 문제가 생기면 이전 상태로 복구
        
- YAML 파일에 “원하는 상태”를 정의하면, 쿠버네티스가 자동으로 맞춤
    

---

## 4. Pod vs ReplicaSet vs Deployment 관계

- **Pod**: 실행 단위
    
- **ReplicaSet**: Pod 개수 보장
    
- **Deployment**: ReplicaSet을 관리하고 업데이트·롤백 제공
    
- 구조: Deployment → ReplicaSet → Pod
    

---

## 5. kubectl 기본 명령어

- Pod 실행 상태 확인
    
    ```bash
    kubectl get pods
    kubectl describe pod <pod-name>
    ```
    
- Deployment 생성/확인
    
    ```bash
    kubectl apply -f deploy.yaml
    kubectl get deployments
    kubectl describe deployment <deploy-name>
    ```
    
- 스케일링
    
    ```bash
    kubectl scale deployment <deploy-name> --replicas=5
    ```
    
- 롤백
    
    ```bash
    kubectl rollout undo deployment <deploy-name>
    ```
    

---

## ✅ 정리

- Pod는 가장 작은 단위, ReplicaSet은 개수 유지, Deployment는 선언형 관리
    
- 실제 운영에서는 Deployment를 통해 Pod를 관리
    
- 다음 주차에서 Service와 Ingress를 통해 외부 연결을 학습
---
# 💻 4주차 실습 강의 – Pod, ReplicaSet, Deployment

## 🎯 실습 목표

- Pod를 직접 생성하고 상태를 확인한다.
    
- ReplicaSet으로 Pod 개수를 유지한다.
    
- Deployment로 Pod를 선언형 방식으로 관리하고, 스케일링·롤링 업데이트·롤백을 실습한다.
    

---

<!-- _class: medium -->

## 1. Pod 생성

### 1-1. Pod 매니페스트 작성 (`nginx-pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
```

### 1-2. 적용

```bash
kubectl apply -f nginx-pod.yaml
```

### 1-3. 확인

```bash
kubectl get pods
kubectl describe pod nginx-pod
```

✅ STATUS가 `Running`이면 성공

---
<!-- _class: medium -->

## 2. ReplicaSet 생성

### 2-1. 매니페스트 작성 (`nginx-rs.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### 2-2. 적용

```bash
kubectl apply -f nginx-rs.yaml
```

---
### 2-3. 확인

```bash
kubectl get rs
kubectl get pods -l app=nginx
```

✅ Pod가 3개 실행 중임을 확인

### 2-4. Pod 삭제 후 확인

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

✅ 삭제된 Pod 대신 새로운 Pod가 자동으로 생성됨

---

<!-- _class: small -->

## 3. Deployment 생성

### 3-1. 매니페스트 작성 (`nginx-deploy.yaml`)

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
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### 3-2. 적용

```bash
kubectl apply -f nginx-deploy.yaml
```

### 3-3. 확인

```bash
kubectl get deployments
kubectl get pods -l app=nginx
```

✅ Deployment와 ReplicaSet이 자동 생성되고 Pod가 실행됨

---

## 4. Deployment 스케일링

```bash
kubectl scale deployment nginx-deploy --replicas=5
kubectl get pods -l app=nginx
```

✅ Pod 개수가 5개로 늘어남

---

## 5. 롤링 업데이트

### 5-1. 새로운 이미지로 업데이트

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.26
```

### 5-2. 진행 확인

```bash
kubectl rollout status deployment/nginx-deploy
```

✅ Pod가 점진적으로 교체됨

---

## 6. 롤백

### 6-1. 롤아웃 기록 확인

```bash
kubectl rollout history deployment/nginx-deploy
```

### 6-2. 롤백 실행

```bash
kubectl rollout undo deployment/nginx-deploy
```

### 6-3. 확인

```bash
kubectl get pods -l app=nginx
```

✅ 이전 버전으로 복구됨

---

## 7. 정리

```bash
kubectl delete -f nginx-pod.yaml
kubectl delete -f nginx-rs.yaml
kubectl delete -f nginx-deploy.yaml
```

---

## ✅ 최종 결과

- Pod 생성과 실행을 이해했다.
    
- ReplicaSet으로 Pod 개수를 유지하는 방법을 익혔다.
    
- Deployment를 통해 스케일링, 롤링 업데이트, 롤백을 실습했다.
    

