
# 🧑‍🏫 12주차 이론 강의 – Trivy를 이용한 컨테이너 이미지 보안 점검

## 1. 컨테이너 이미지 보안의 필요성

- 컨테이너 이미지는 다양한 라이브러리와 패키지를 포함
    
- 알려진 취약점(CVE)이 포함될 경우 공격에 노출
    
- 실무에서는 배포 전 반드시 **이미지 보안 점검** 필수
    
- DevSecOps의 핵심: **Shift Left Security** (개발 단계부터 보안 내재화)
    

---

## 2. Trivy 개요

- 오픈소스 보안 스캐너 (Aqua Security 개발)
    
- **경량, 빠른 속도, 설치 간편**
    
- 주요 기능:
    
    1. **이미지 스캐닝**: 컨테이너 이미지 취약점 점검
        
    2. **파일 시스템/코드 스캐닝**: 로컬 프로젝트 취약점 점검
        
    3. **IaC 스캐닝**: Terraform, Kubernetes YAML 보안 설정 점검
        
    4. **클라우드 계정 점검**: AWS, GCP, Azure 설정 보안 검증
        

---

## 3. Trivy 아키텍처

- **취약점 DB**: NVD, Red Hat, Debian, GitHub Advisory 등 다양한 소스 활용
    
- **로컬 캐시**: 처음 실행 시 DB 다운로드, 이후 증분 업데이트
    
- **출력 형식**: Table, JSON, SARIF(GitHub Security Tab 지원)
    

---

## 4. 컨테이너 이미지 스캔 흐름

1. 개발자가 Dockerfile로 이미지 빌드
    
2. Trivy 실행 → 이미지 내 패키지 분석
    
3. 패키지 버전과 취약점 DB 비교
    
4. CVE 목록, 심각도(High, Medium, Low) 보고
    
5. 보고서(JSON, Markdown, SARIF) 저장 → CI/CD 파이프라인에 통합
    

---

## 5. Trivy 사용 예시

### 이미지 취약점 스캔

```bash
trivy image nginx:1.25
```

출력 예시:

```
nginx:1.25 (debian 11.6)
=========================
Total: 15 (HIGH: 5, MEDIUM: 7, LOW: 3)

CVE-2023-12345 | HIGH | OpenSSL | Fixed in: 1.1.1u
CVE-2023-67890 | MEDIUM | glibc  | Fixed in: 2.31-13+deb11u5
...
```

---
### 파일 시스템 스캔

```bash
trivy fs .
```

### IaC (Terraform) 스캔

```bash
trivy config ./terraform
```

### 클라우드 계정 점검

```bash
trivy aws --region ap-northeast-2
```

---

## 6. CI/CD와의 통합

- GitHub Actions, GitLab CI, Jenkins 등에서 자동 실행 가능
    
- `.trivyignore` 파일을 통해 False Positive 관리
    
- SARIF 포맷으로 변환하여 GitHub Security 탭에 자동 표시
    

---

## 7. Trivy 장점

- 오픈소스이면서도 기업 환경에서도 널리 사용
    
- 설치 및 사용이 단순 → 교육 현장에서 빠르게 적용 가능
    
- CSPM(Cloud Security Posture Management) + CWPP(Container Security) 기능 통합
    

---

## ✅ 정리

- Trivy = 컨테이너 이미지 및 IaC 보안 점검 도구
    
- DevSecOps 환경에서 **배포 전 보안 검증 자동화** 가능
    
- CI/CD 파이프라인에 쉽게 통합 가능
    
- 13주차에서는 Gatekeeper와 Falco를 활용하여 **실행 중인 클러스터 정책 위반과 런타임 보안 이벤트 탐지**를 학습
    

---

# 💻 12주차 실습 강의 – Trivy를 이용한 컨테이너 이미지 보안 점검

## 🎯 실습 목표

- Trivy를 설치하고 동작을 확인한다.
    
- 컨테이너 이미지를 스캔하여 취약점을 분석한다.
    
- 프로젝트 디렉토리와 IaC(Terraform) 설정 파일을 점검한다.
    
- GitHub Actions에 Trivy를 연동하는 기초를 경험한다.
    

---

## 1. Trivy 설치

### 1-1. Linux / Mac

```bash
brew install aquasecurity/trivy/trivy
```

또는

```bash
sudo apt-get install wget -y
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.0_Linux-64bit.deb
sudo dpkg -i trivy_0.50.0_Linux-64bit.deb
```

### 1-2. 버전 확인

```bash
trivy --version
```

✅ `Version: 0.50.0` 같은 결과가 출력되면 성공

---

## 2. 컨테이너 이미지 스캔

### 2-1. nginx 최신 버전 점검

```bash
trivy image nginx:latest
```

✅ 출력 예시

```
nginx:latest (debian 11)
========================
Total: 20 (HIGH: 5, MEDIUM: 10, LOW: 5)

CVE-2023-12345 | HIGH | openssl | Fixed in: 1.1.1u
CVE-2022-56789 | MEDIUM | zlib   | Fixed in: 1.2.11.dfsg-2+deb11u2
...
```

### 2-2. JSON 리포트 저장

```bash
trivy image -f json -o nginx-report.json nginx:latest
```

---

## 3. 파일 시스템 점검

### 3-1. 현재 디렉토리 스캔

```bash
trivy fs .
```

✅ Node.js 프로젝트라면 `package-lock.json` 기반 취약점 확인

---

## 4. IaC 설정 점검 (Terraform 코드)

### 4-1. Terraform 디렉토리 점검

```bash
trivy config ./terraform
```

✅ 출력 예시

```
terraform/main.tf
-----------------
WARN: Security group allows 0.0.0.0/0 on port 22 (potential risk)
```

---

## 5. 클라우드 계정 점검 (선택)

### AWS 계정 점검

```bash
trivy aws --region ap-northeast-2
```

✅ IAM 정책, S3 버킷 퍼블릭 접근 여부 등 보안 점검 결과 출력

---
<!-- _class: medium -->

## 6. GitHub Actions 연동 (기초)

### 6-1. Workflow 파일 생성 `.github/workflows/trivy-scan.yaml`

```yaml
name: Trivy Scan

on: [push]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run Trivy on Docker Image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'nginx:latest'
          format: 'table'
```

### 6-2. 실행 확인

- GitHub 저장소 → **Actions 탭** → Trivy Scan 결과 확인
    

---

## 7. 정리

- 컨테이너 이미지 스캔
    
- 파일 시스템, IaC 코드 보안 점검
    
- CI/CD와 연계한 자동 보안 스캐닝 체험
    

---

## ✅ 최종 결과

- Trivy 설치 및 실행 성공
    
- nginx 이미지 보안 점검 결과 확인
    
- Terraform 코드의 보안 취약점 탐지
    
- GitHub Actions 연동으로 DevSecOps 파이프라인 기반 확보
    
