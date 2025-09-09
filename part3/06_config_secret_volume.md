# 🧑‍🏫 6주차 이론 강의 – ConfigMap, Secret, Volume

## 1. Kubernetes에서 설정과 데이터의 필요성

- 애플리케이션 실행 시 **환경설정, 비밀번호, 데이터 저장소** 필요
    
- Pod 안에 직접 값을 넣으면 → 이미지 수정 필요, 보안 취약
    
- Kubernetes는 이를 해결하기 위해 **ConfigMap, Secret, Volume** 제공
    

---

## 2. ConfigMap

- **환경 설정값**을 Key-Value 형태로 저장
    
- Pod/컨테이너에 주입 가능 (환경변수, 설정파일)
    
- 장점: 이미지와 설정 분리 → 동일 이미지 여러 환경에서 사용 가능
    

### 예시: ConfigMap 정의

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  APP_COLOR: "blue"
```
---
### Pod에서 사용 (환경변수로 주입)

```yaml
spec:
  containers:
  - name: myapp
    image: busybox
    env:
    - name: MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_MODE
```

---

## 3. Secret

- **민감한 데이터(비밀번호, 토큰, 인증서)** 저장
    
- Base64 인코딩 방식으로 저장
    
- Pod 내 환경변수 또는 Volume으로 주입 가능
    

### 예시: Secret 정의

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=   # base64(admin)
  password: cGFzc3dvcmQ= # base64(password)
```
---
### Pod에서 사용 (환경변수)

```yaml
spec:
  containers:
  - name: mydb
    image: mysql
    env:
    - name: MYSQL_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
```

---

## 4. Volume

- Pod 안에서 사용하는 **스토리지 영역**
    
- 컨테이너가 죽어도 데이터 유지 가능 (Persistent Volume 사용 시)
    

### Volume 종류

1. **emptyDir**: Pod가 살아있는 동안만 유지, 임시 데이터 저장용
    
2. **hostPath**: 노드 파일시스템 경로와 연결
    
3. **PersistentVolume (PV)**: 클러스터 차원의 스토리지 (NFS, EBS 등)
    
4. **PersistentVolumeClaim (PVC)**: Pod가 PV를 요청하는 방식
    
---
### 예시: emptyDir Volume

```yaml
spec:
  containers:
  - name: cache
    image: busybox
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

---

## 5. ConfigMap, Secret, Volume 관계

- ConfigMap: 일반 환경설정
    
- Secret: 민감정보
    
- Volume: 데이터 저장소
    
- **실무 예시**:
    
    - ConfigMap → 앱 실행 모드(dev/prod)
        
    - Secret → DB 계정 정보
        
    - Volume → DB 데이터 저장소
        

---

## ✅ 정리

- **ConfigMap**: 일반 설정값 저장, Pod에 환경변수/파일로 전달
    
- **Secret**: 비밀번호/토큰 같은 민감정보 저장, 보안 강화
    
- **Volume**: 데이터 영속성 제공, Pod 종료 시에도 데이터 유지 가능
    
- Kubernetes는 애플리케이션 실행에 필요한 **설정, 보안, 데이터 저장**을 체계적으로 관리
    

---

# 💻 6주차 실습 강의 – ConfigMap, Secret, Volume

## 🎯 실습 목표

- ConfigMap을 생성하고 Pod에 환경변수로 주입한다.
    
- Secret을 생성하고 Pod에서 비밀번호를 안전하게 사용한다.
    
- Volume을 사용하여 데이터 저장소를 유지한다.
    

---

## 1. ConfigMap 실습

### 1-1. ConfigMap 생성 (`app-config.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  APP_COLOR: "blue"
```

```bash
kubectl apply -f app-config.yaml
kubectl get configmap app-config -o yaml
```

---
<!-- _class: medium -->

### 1-2. Pod에서 ConfigMap 사용 (`pod-with-config.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: demo-container
    image: busybox
    command: ["sh", "-c", "echo Mode=$APP_MODE, Color=$APP_COLOR && sleep 3600"]
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_MODE
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

```bash
kubectl apply -f pod-with-config.yaml
kubectl logs configmap-demo
```

✅ 출력: `Mode=production, Color=blue`

---

## 2. Secret 실습

### 2-1. Secret 생성 (`db-secret.yaml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=      # base64(admin)
  password: cGFzc3dvcmQ=  # base64(password)
```

```bash
kubectl apply -f db-secret.yaml
kubectl get secret db-secret -o yaml
```

---
<!-- _class: medium -->

### 2-2. Pod에서 Secret 사용 (`pod-with-secret.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: demo-container
    image: busybox
    command: ["sh", "-c", "echo User=$MYSQL_USER, Pass=$MYSQL_PASS && sleep 3600"]
    env:
    - name: MYSQL_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: MYSQL_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

```bash
kubectl apply -f pod-with-secret.yaml
kubectl logs secret-demo
```

✅ 출력: `User=admin, Pass=password`

---

<!-- _class: medium -->

## 3. Volume 실습

### 3-1. emptyDir Volume (`pod-with-volume.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo Hello > /data/hello.txt && sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data
  - name: reader
    image: busybox
    command: ["sh", "-c", "cat /data/hello.txt && sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    emptyDir: {}
```

```bash
kubectl apply -f pod-with-volume.yaml
kubectl logs volume-demo -c reader
```

✅ 출력: `Hello`

---

## 4. 정리

```bash
kubectl delete -f pod-with-config.yaml
kubectl delete -f pod-with-secret.yaml
kubectl delete -f pod-with-volume.yaml
kubectl delete -f app-config.yaml
kubectl delete -f db-secret.yaml
```

---

## ✅ 최종 결과

- ConfigMap을 이용해 설정값을 Pod에 전달
    
- Secret으로 민감 정보를 안전하게 관리
    
- Volume을 통해 Pod 간 데이터 공유 및 유지
    
