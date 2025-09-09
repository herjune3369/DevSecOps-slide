
# 🧑‍🏫 10주차 이론 강의 – Ansible을 활용한 Kubernetes 앱 배포 자동화

## 1. 왜 Ansible인가?

- Kubernetes 클러스터(EKS 등)가 준비된 후에도 **애플리케이션 배포와 설정**은 반복적으로 발생
    
- 수동으로 `kubectl apply` 하거나 매번 YAML을 수정 → **비효율적**
    
- Ansible은 **자동화된 배포 도구**로, Kubernetes 리소스를 코드로 관리 가능
    

---

## 2. Ansible 개요

- 오픈소스 **IT 자동화 도구**
    
- 대상 서버에 Agent 설치 불필요 (**Agentless**, SSH 사용)
    
- **YAML 기반 Playbook**을 사용 → 사람이 읽기 쉬움
    
- Terraform이 **인프라 레벨 자동화**라면, Ansible은 **애플리케이션 레벨 자동화**
    

---

## 3. Ansible 기본 구성 요소

### Inventory

- 관리할 대상 정의 (서버, 클러스터, 그룹)
    

```ini
[k8s-master]
192.168.10.10

[k8s-workers]
192.168.10.11
192.168.10.12
```

---
### Playbook

- 실행할 작업(Task) 집합
    

```yaml
- name: Deploy app to Kubernetes
  hosts: k8s-master
  tasks:
    - name: Apply Deployment
      kubernetes.core.k8s:
        state: present
        src: ./manifests/deployment.yaml
```

### Roles

- Playbook을 모듈화한 구조 (tasks, templates, vars 등 디렉토리 단위)
    
- 재사용성과 유지보수성 향상
    

---

## 4. Ansible과 Kubernetes

- **kubernetes.core.k8s 모듈**을 통해 YAML 매니페스트 적용 가능
    
- Helm과 연계하여 Chart 배포 가능
    
- GitHub Actions 같은 CI/CD 파이프라인과 통합 시 자동 배포 파이프라인 구축
    

---

## 5. Ansible vs kubectl 직접 배포

|구분|kubectl 수동 배포|Ansible 자동화|
|---|---|---|
|실행 방식|사람이 명령 입력|코드 기반 자동 실행|
|반복 작업|동일 YAML 매번 적용|Playbook 한 번 실행|
|확장성|서버 많아지면 불편|수백 대 서버에도 동일하게 적용|
|유지보수|YAML 파일 흩어짐|Role 구조로 체계적 관리|

---

## 6. Ansible로 Kubernetes 앱 배포 흐름

1. Inventory 작성 (EKS Cluster 정보)
    
2. Playbook 정의 (Deployment, Service, ConfigMap 등)
    
3. `ansible-playbook` 실행 → Kubernetes API 호출
    
4. 앱 자동 배포 및 상태 확인
    

---

## 7. 실무 활용 예시

- **웹 서비스 자동 배포**: Nginx, Flask, Node.js 앱
    
- **데이터베이스 초기화**: MySQL, PostgreSQL 등 DB 설정 자동화
    
- **보안 정책 적용**: Namespace, RBAC 설정 자동화
    
- **CI/CD 통합**: GitHub Actions → Ansible → Kubernetes 자동 배포
    

---

## ✅ 정리

- Terraform = **클라우드 인프라 생성**
    
- Ansible = **Kubernetes 앱 배포/운영 자동화**
    
- Playbook과 Role 구조로 반복 작업을 자동화하고, CI/CD와 연계 가능
    
- 다음(11주차)에서는 GitHub Actions로 CI/CD 워크플로우를 직접 작성
    

---

# 💻 10주차 실습 강의 – Ansible을 활용한 Kubernetes 앱 배포 자동화

## 🎯 실습 목표

- Ansible 설치 및 환경 준비
    
- Inventory, Playbook 작성
    
- Ansible `kubernetes.core.k8s` 모듈로 Deployment, Service 자동 배포
    
- Ansible Role 구조 체험
    

---

## 1. 환경 준비

### 1-1. Ansible 설치

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
```

✅ `ansible [core X.Y.Z]` 출력

### 1-2. Kubernetes Ansible 모듈 설치

```bash
ansible-galaxy collection install kubernetes.core
pip install openshift
```

---
### 1-3. `~/.kube/config` 확인

```bash
kubectl get nodes
```

✅ EKS 클러스터 연결 정상

---

## 2. Inventory 작성

### 2-1. `inventory.ini`

```ini
[localhost]
127.0.0.1 ansible_connection=local
```

👉 EKS는 API Server로 접근하기 때문에 로컬에서 실행

---

## 3. Playbook 작성

### 3-1. Deployment 매니페스트 (`manifests/deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
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
```
---
### 3-2. Service 매니페스트 (`manifests/service.yaml`)

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
  type: LoadBalancer
```

---
### 3-3. Ansible Playbook (`deploy-app.yaml`)

```yaml
- name: Deploy Application to Kubernetes
  hosts: localhost
  tasks:
    - name: Apply Deployment
      kubernetes.core.k8s:
        state: present
        src: ./manifests/deployment.yaml

    - name: Apply Service
      kubernetes.core.k8s:
        state: present
        src: ./manifests/service.yaml
```

---

## 4. Playbook 실행

```bash
ansible-playbook -i inventory.ini deploy-app.yaml
```

✅ 출력 예시:

```
TASK [Apply Deployment] ***
changed: [127.0.0.1]

TASK [Apply Service] ***
changed: [127.0.0.1]

PLAY RECAP ***
127.0.0.1 : ok=2 changed=2
```

---

## 5. 배포 확인

```bash
kubectl get deployments
kubectl get pods -l app=myapp
kubectl get svc myapp-svc
```

✅ `EXTERNAL-IP` 할당 → 브라우저 접속 시 nginx 페이지 확인

---

## 6. Role 구조 적용 (선택)

### 6-1. 디렉토리 구조

```
roles/
 └── k8s-app/
     ├── tasks/main.yml
     ├── files/
     └── templates/
```

### 6-2. Role Task (`roles/k8s-app/tasks/main.yml`)

```yaml
- name: Apply Deployment
  kubernetes.core.k8s:
    state: present
    src: ./manifests/deployment.yaml

- name: Apply Service
  kubernetes.core.k8s:
    state: present
    src: ./manifests/service.yaml
```
---
### 6-3. Playbook 수정 (`site.yaml`)

```yaml
- name: Deploy app using role
  hosts: localhost
  roles:
    - k8s-app
```

실행:

```bash
ansible-playbook -i inventory.ini site.yaml
```

---

## 7. 정리

```bash
kubectl delete -f manifests/service.yaml
kubectl delete -f manifests/deployment.yaml
```

---

## ✅ 최종 결과

- Ansible로 Kubernetes 리소스를 자동 배포하는 방법 학습
    
- Deployment + Service 생성 → 브라우저 접속 확인
    
- Role 구조를 통해 유지보수성과 재사용성 강화
    
