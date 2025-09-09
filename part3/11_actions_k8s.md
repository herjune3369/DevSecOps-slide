
# 🧑‍🏫 11주차 이론 강의 – GitHub Actions CI/CD 개요와 워크플로우 작성

## 1. CI/CD란?

- **CI (Continuous Integration, 지속적 통합)**
    
    - 개발자가 코드를 push할 때마다 자동으로 빌드·테스트 실행
        
- **CD (Continuous Delivery/Deployment, 지속적 배포)**
    
    - 빌드된 애플리케이션을 자동으로 배포 (테스트 서버 → 운영 서버)
        
- 목표: **코드 변경 → 테스트 → 배포** 전체 과정을 자동화
    

---

## 2. GitHub Actions 개요

- GitHub에서 제공하는 CI/CD 플랫폼
    
- 이벤트 기반으로 워크플로우 실행
    
- 주요 특징:
    
    - GitHub 저장소와 자연스럽게 연동
        
    - Docker, Kubernetes, AWS 등 다양한 환경에 배포 가능
        
    - Marketplace에서 수천 개의 Actions(모듈) 제공
        

---
<!-- _class: medium -->

## 3. GitHub Actions 기본 구조

### Workflow 파일 (`.github/workflows/deploy.yaml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Run Tests
        run: echo "Running unit tests..."

      - name: Build Docker Image
        run: docker build -t myapp:latest .

      - name: Deploy to Kubernetes
        run: kubectl apply -f k8s/deployment.yaml
```

---

## 4. GitHub Actions 구성 요소

- **Workflow**
    
    - 자동화 전체 과정 (YAML 파일로 정의)
        
- **Event**
    
    - Workflow 실행 트리거 (예: push, pull_request, schedule)
        
- **Job**
    
    - 실행 단위, 여러 Step으로 구성
        
- **Step**
    
    - 개별 작업 (커맨드 실행 또는 Action 사용)
        
- **Action**
    
    - 재사용 가능한 작업 모듈 (ex: checkout, setup-node, aws-actions 등)
        

---

## 5. CI/CD 일반적인 흐름

1. 개발자가 GitHub 저장소에 push
    
2. GitHub Actions가 자동 실행
    
3. 코드 빌드 및 단위 테스트
    
4. Docker 이미지 빌드 & 레지스트리 푸시(ECR, DockerHub 등)
    
5. Kubernetes 매니페스트 적용 (kubectl)
    
6. 성공/실패 알림
    

---

## 6. GitHub Actions의 장점

- GitHub에 코드만 올리면 바로 CI/CD 환경 구성 가능
    
- 무료 사용량 제공 (GitHub Free 계정도 사용 가능)
    
- AWS, GCP, Azure 등 클라우드 서비스와 손쉽게 연동
    
- 보안 관리 (Secrets에 AWS Key, Token 저장)
    

---

## 7. Kubernetes와 GitHub Actions 통합

- Terraform으로 만든 EKS 클러스터에 접근하기 위해 **AWS IAM Key**와 `kubeconfig` 필요
    
- GitHub Secrets에 등록
    
    - `AWS_ACCESS_KEY_ID`
        
    - `AWS_SECRET_ACCESS_KEY`
        
    - `AWS_REGION`
        
- Workflow에서 AWS 인증 후 `kubectl` 명령 실행
    

---

## ✅ 정리

- CI/CD = 코드 변경부터 배포까지 자동화하는 DevOps 핵심 개념
    
- GitHub Actions = GitHub 기반 CI/CD 플랫폼, 이벤트 기반 Workflow 실행
    
- Job, Step, Action 개념 이해 필수
    
- Kubernetes와 연결하여 **자동 배포 파이프라인** 구축 가능
    
- 다음(12주차)에서는 **Trivy를 이용한 컨테이너 이미지 보안 점검**을 학습
    

---

# 💻 11주차 실습 강의 – GitHub Actions CI/CD 워크플로우 작성

## 🎯 실습 목표

- GitHub Actions Workflow를 작성하고 실행한다.
    
- 코드 변경 시 자동 빌드·테스트·배포가 동작하는 CI/CD 파이프라인을 체험한다.
    
- AWS EKS와 연계하여 Kubernetes에 자동 배포한다.
    

---

## 1. 사전 준비

### 1-1. GitHub 저장소 생성

- GitHub에서 새 저장소(`k8s-cicd-demo`) 생성
    
- 로컬에서 초기화 후 push
    

```bash
git init
git remote add origin https://github.com/<username>/k8s-cicd-demo.git
git add .
git commit -m "init"
git push origin main
```

---
### 1-2. GitHub Secrets 등록

저장소 → **Settings → Secrets and variables → Actions → New repository secret**

- `AWS_ACCESS_KEY_ID`
    
- `AWS_SECRET_ACCESS_KEY`
    
- `AWS_REGION` = `ap-northeast-2`
    
- `EKS_CLUSTER_NAME` = `eks-demo`
    

---

## 2. Kubernetes 매니페스트 작성

### 2-1. `k8s/deployment.yaml`

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
        image: <AWS_ACCOUNT_ID>.dkr.ecr.ap-northeast-2.amazonaws.com/myapp:latest
        ports:
        - containerPort: 80
```
---
### 2-2. `k8s/service.yaml`

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
<!-- _class: small -->

## 3. GitHub Actions Workflow 작성

### 3-1. `.github/workflows/deploy.yaml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. 저장소 체크아웃
      - name: Checkout
        uses: actions/checkout@v3

      # 2. AWS 인증
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      # 3. ECR 로그인
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      # 4. Docker 이미지 빌드 & 푸시
      - name: Build and Push Docker image
        run: |
          IMAGE_URI=${{ steps.login-ecr.outputs.registry }}/myapp:latest
          docker build -t $IMAGE_URI .
          docker push $IMAGE_URI

      # 5. kubeconfig 설정
      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig --region ${{ secrets.AWS_REGION }} --name ${{ secrets.EKS_CLUSTER_NAME }}

      # 6. Kubernetes 배포
      - name: Deploy to EKS
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
```

---

## 4. Workflow 실행

1. 로컬에서 코드 변경 후 push
    

```bash
git add .
git commit -m "add CI/CD workflow"
git push origin main
```

2. GitHub → **Actions 탭** 이동 → 워크플로우 실행 확인
    
    - Checkout → Build → Docker Push → kubectl apply 단계 순서대로 진행
        
    - 성공 시 초록색 체크 표시
        

---

## 5. 배포 확인

```bash
kubectl get deployments
kubectl get pods -l app=myapp
kubectl get svc myapp-svc
```

✅ `EXTERNAL-IP` 할당 후 브라우저 접속 → nginx 페이지 출력

---

## 6. 정리

- GitHub 저장소 삭제 시 Workflow도 제거됨
    
- Kubernetes 리소스 삭제
    

```bash
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml
```

---

## ✅ 최종 결과

- GitHub Actions를 통해 **코드 변경 → 이미지 빌드/푸시 → EKS 배포** 전체 흐름 자동화
    
- 학생들은 실제 **CI/CD 파이프라인 작동 원리**를 체험
    
