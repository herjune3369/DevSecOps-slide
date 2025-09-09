

# 📘 클라우드 네이티브 환경에서 배우는 Kubernetes 운영과 실전 배포

## Part 1. Kubernetes 기본기 다지기 – 배포와 운영
##  Part 2. 실전 DevOps 워크숍 – Terraform & Ansible 기반 Kubernetes 배포

---

## 🎯 Part 1 학습목표 

- **쿠버네티스 기본 개념과 아키텍처 이해**
    
- **Pod, Deployment, Service, Ingress 등 핵심 리소스 운영 능력 습득**
    
- **ConfigMap, Secret, Volume으로 앱 설정 관리**
    
- **Helm을 활용한 애플리케이션 배포 자동화 경험**
    
- **단순 웹 애플리케이션을 쿠버네티스에 배포·운영할 수 있는 실습 능력 확보**

---
## 📚 Part 1. Kubernetes 기본기 다지기 – 배포와 운영 (1~8주차)

1주: 클라우드 네이티브와 Kubernetes 개요 이해  
2주: Docker 컨테이너 기본 개념과 실행·네트워크·볼륨 학습  
3주: Kubernetes 아키텍처와 kubectl 기본 사용법 습득  
4주: Pod, ReplicaSet, Deployment를 통한 앱 배포와 스케일링 실습  
5주: Service와 Ingress로 외부 접근 구성 경험  
6주: ConfigMap, Secret, Volume을 활용한 설정 및 데이터 관리  
7주: Helm Chart로 애플리케이션 배포 자동화  
8주: Mini Project – 단순 웹앱을 Kubernetes에 배포하고 운영

---
## 🎯 Part 2 학습목표

**실무형 자동화 도구를 활용해 Kubernetes 환경을 배포·운영할 수 있는 능력 확보**

- **Terraform**으로 클라우드 인프라(EKS 등) 자동 구성
    
- **Ansible**로 애플리케이션 배포 자동화
    
- **GitHub Actions**로 CI/CD 파이프라인 구축
    
- **Trivy 등 일부 보안 도구**를 통한 이미지·구성 점검 경험
    
- **실전 프로젝트 수행**을 통해 DevOps·DevSecOps 워크플로우 전체 흐름 이해

---
## 📚 Part 2. **클라우드 보안형 DevOps 파이프라인 실습 – Kubernetes 배포와 DevSecOps 자동화 (9~15주차)**

9주: Terraform 기본 문법과 클라우드 인프라(EKS 등) 배포  
10주: Ansible을 활용한 Kubernetes 앱 배포 자동화  
11주: GitHub Actions CI/CD 개요와 워크플로우 작성  
12주: Trivy를 이용한 컨테이너 이미지 보안 점검  
13주: Gatekeeper 정책 위반 탐지와 Falco 런타임 보안 이벤트 분석  
14주: 보안 리포트 자동 생성 스크립트 활용 (generate_security_report.py)  
15주: Mini Project – Terraform+Ansible+CI/CD+보안 점검을 포함한 실전 배포 파이프라인 완성

---
# Part 1. Kubernetes 기본기 다지기 – 배포와 운영


---

# 🧑‍🏫 1주차 이론 강의 – 클라우드 & Kubernetes 개요

## 🎯 학습 목표

- 클라우드 네이티브와 DevOps 개념을 이해한다.
    
- 컨테이너와 쿠버네티스의 필요성을 설명할 수 있다.
    
- 쿠버네티스 아키텍처 개요를 파악한다.
    

---

## 1. 클라우드 네이티브란?

- **모놀리식 아키텍처 → 마이크로서비스 아키텍처(MSA)**
    
- **특징**: 확장성(Scalability), 이식성(Portability), 자동화(Automation)
    
- **예시**: Netflix, 쿠팡, 배달의민족 → 하루 수백만 건 트래픽을 안정적으로 처리
    

---

## 2. DevOps 개념

- **Dev(개발) + Ops(운영)** 결합
    
- 목표: 빠른 배포, 안정적 운영, 지속적인 개선
    
- 주요 개념
    
    - **CI (Continuous Integration)**: 코드 변경 자동 빌드·테스트
        
    - **CD (Continuous Delivery/Deployment)**: 자동 배포
        
    - **IaC (Infrastructure as Code)**: 인프라를 코드로 관리
        

---

## 3. 컨테이너의 필요성

- 기존: VM(가상머신) → 무겁고 느림
    
- Docker 컨테이너: 가볍고 빠른 실행, 일관된 환경 제공
    
- 예시: 개발 PC와 서버 환경 차이로 발생하는 **“Works on my machine” 문제 해결**
    

---

## 4. Kubernetes 등장 배경

- 컨테이너가 늘어나면 → 관리 어려움 (수십~수백 개 컨테이너)
    
- 필요 기능
    
    - 컨테이너 자동 배포 및 스케줄링
        
    - 서비스 디스커버리와 로드밸런싱
        
    - 장애 시 자동 복구(Self-healing)
        
    - 확장(Scaling) 자동화
        

👉 이를 해결하기 위해 Google이 개발 → 오픈소스로 공개

---

## 5. Kubernetes 아키텍처 개요

- **Control Plane (두뇌 역할)**
    
    - API Server: 명령 수신
        
    - etcd: 상태 저장 DB
        
    - Scheduler: Pod 배치
        
    - Controller Manager: 상태 관리
        
- **Worker Node (작업자 역할)**
    
    - kubelet: Pod 실행 관리
        
    - kube-proxy: 네트워크 라우팅
        
    - Container Runtime: Docker/Containerd
        

---

## 6. Kubernetes 활용 사례

- **대규모 서비스 운영**: Google, Spotify, 쿠팡 → 수천 개 컨테이너 관리
    
- **클라우드 서비스**: AWS EKS, GCP GKE, Azure AKS → 관리형 Kubernetes
    
- **기업 채택 이유**: 확장성, 비용 최적화, 자동화
    

---

## ✅ 정리

- 클라우드 네이티브 시대에는 컨테이너 기반 운영이 필수
    
- Kubernetes는 컨테이너를 자동 관리하는 **표준 플랫폼**
    
- 2주차부터는 Docker 기초 → Kubernetes 실습으로 진행
    


---

# 💻 1주차 실습 강의 – Hello Kubernetes

## 🎯 실습 목표

- 로컬 환경에서 **Kubernetes 클러스터(Minikube)**를 실행한다.
    
- `kubectl`을 사용하여 **Deployment**와 **Service**를 생성한다.
    
- 브라우저에서 **Hello Kubernetes** 애플리케이션에 접속한다.
    

---

<!-- _class: medium-->


## 1. 실습 환경 준비

### 1-1. 필수 소프트웨어 설치

아래 프로그램이 설치되어 있어야 한다.

1. **Docker Desktop** (또는 Docker Engine)  
    👉 컨테이너 실행을 위한 필수 프로그램  
    [Docker 설치 링크](https://docs.docker.com/get-docker/)
    
2. **kubectl** (쿠버네티스 CLI)  
    👉 쿠버네티스 클러스터와 상호작용하기 위한 도구  
    설치 확인:
    
    ```bash
    kubectl version --client
    ```
    
    정상 설치 시 버전 정보가 출력됨.
    
3. **Minikube**  
    👉 로컬에서 Kubernetes 클러스터를 실행하는 툴  
    설치 확인:
    
    ```bash
    minikube version
    ```
    
    정상 설치 시 버전 정보가 출력됨.
    

---

<!-- _class: medium -->

## 2. Minikube 클러스터 실행

1. Minikube 클러스터 시작:
    
    ```bash
    minikube start --driver=docker
    ```
    
    - `--driver=docker` : Docker를 사용하여 클러스터 실행
        
    - 처음 실행 시 이미지 다운로드 때문에 몇 분 정도 걸림
        
2. 클러스터 상태 확인:
    
    ```bash
    minikube status
    ```
    
    ✅ `host: Running`, `kubelet: Running`, `apiserver: Running` 등이 나오면 정상 실행
    
3. 노드 확인:
    
    ```bash
    kubectl get nodes
    ```
    
    ✅ `minikube Ready` 표시되면 클러스터 준비 완료
    

---

## 3. Hello Kubernetes 애플리케이션 배포

### 3-1. Deployment 생성

```bash
kubectl create deployment hello-k8s \
  --image=k8s.gcr.io/echoserver:1.4
```

- `deployment.apps/hello-k8s created` 출력되면 정상 생성
    

확인:

```bash
kubectl get deployments
kubectl get pods
```

✅ `hello-k8s` Deployment와 Pod가 `Running` 상태면 성공

---

### 3-2. Service로 외부 접속 열기

```bash
kubectl expose deployment hello-k8s \
  --type=NodePort \
  --port=8080
```

- `service/hello-k8s exposed` 출력
    

확인:

```bash
kubectl get svc
```

✅ `hello-k8s` 서비스가 `NodePort` 타입으로 생성됨

---

## 4. 애플리케이션 접속

1. 서비스 URL 확인:
    
    ```bash
    minikube service hello-k8s --url
    ```
    
    예시 출력:
    
    ```
    http://127.0.0.1:50321
    ```
    
2. 웹 브라우저에서 위 주소 접속  
    → `Hello Kubernetes!` 와 유사한 응답 화면 출력
    

---

## 5. 클린업 (정리)

실습 종료 후 리소스를 정리한다.

1. 서비스 삭제
    
    ```bash
    kubectl delete svc hello-k8s
    ```
    
2. Deployment 삭제
    
    ```bash
    kubectl delete deployment hello-k8s
    ```
    
3. Minikube 종료
    
    ```bash
    minikube stop
    ```
    

---

## ✅ 최종 결과

- 학생들은 `kubectl`을 사용해 Deployment와 Service를 만들고,
    
- **브라우저에서 Hello Kubernetes 애플리케이션에 접속**하는 경험을 한다.
    

---
