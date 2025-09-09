
# 2주차: Trivy 이론 – 오픈소스 보안 스캐너의 역할과 원리

---

## 🎯 학습 목표

- Trivy가 무엇인지, 어떤 원리로 동작하는지 설명할 수 있다
    
- Trivy가 지원하는 보안 점검 범위를 이해한다
    
- Trivy의 장단점과 활용 시나리오를 파악한다
    

---

## 1️⃣ Trivy란 무엇인가?

- **Trivy (Aqua Security)**: 오픈소스 보안 스캐너
    
- 이름 의미: _Tri_ (세 가지) + _V_ (Vulnerability) → 취약점 점검 도구
    
- 단일 바이너리로 동작 → 설치와 사용이 간단
    
- DevSecOps 파이프라인에 쉽게 통합 가능
    

---

## 2️⃣ Trivy가 점검하는 대상

1. **컨테이너 이미지 (CWPP)**
    
    - 예: `python:3.9`, `nginx:latest`
        
    - OS 패키지 + 언어별 라이브러리 취약점
        
2. **파일시스템/로컬 디렉토리**
    
    - 서버나 프로젝트 폴더 내 취약한 의존성
        
3. **IaC (Infrastructure as Code) – CSPM**
    
    - Terraform, Kubernetes YAML
        
    - 보안 그룹 과다 개방, 하드코딩된 비밀번호 등 탐지
        
4. **클라우드 계정 – CSPM**
    
    - AWS 계정 설정 (IAM, S3, RDS, VPC 등)
        

---

## 3️⃣ Trivy의 동작 원리

1. **취약점 DB 다운로드**
    
    - GitHub Security Advisory, NVD, Red Hat 등에서 데이터 수집
        
    - 로컬 캐시 저장, 주기적 업데이트
        
2. **스캔 프로세스**
    
    - 분석 대상(OS, 패키지, IaC 코드, 클라우드 설정) 식별
        
    - DB와 매칭 → CVE ID, 심각도(Critical/High/Medium/Low) 표시
        
3. **출력 형식**
    
    - 표 (CLI 기본)
        
    - JSON (자동화 활용)
        
    - SARIF (GitHub Security Tab)
        
    - Markdown/PDF (리포트용)
        

---

## 4️⃣ Trivy의 장점과 한계

### 장점

- 설치 간단, 빠른 동작 속도
    
- 다양한 스캔 모드 지원 (image/config/fs/aws)
    
- 오픈소스 → 무료 사용 가능
    
- GitHub Actions, Jenkins, GitLab CI 등과 쉽게 연동
    

### 한계

- 런타임 보호 기능은 제한적 (Falco 같은 도구 필요)
    
- False Positive 가능성 존재 → `.trivyignore`로 관리 필요
    
- DB 업데이트 속도에 따라 최신 취약점 반영 지연 가능
    

---

## 5️⃣ Trivy의 DevSecOps 활용 시나리오

1. **개발 단계**
    
    - 개발자가 로컬에서 `trivy image` 실행
        
    - 문제된 패키지를 수정 후 커밋
        
2. **CI/CD 단계**
    
    - GitHub Actions Workflow에 Trivy Step 추가
        
    - 배포 전 자동 취약점 점검
        
3. **운영 단계**
    
    - 주기적으로 `trivy aws` 실행 → 계정 보안 상태 점검
        
    - IaC 코드 변경 시 `trivy config`로 검증
        

---

## 6️⃣ 사례

- **금융권**: Trivy를 GitHub Actions에 연동 → Docker 이미지 스캔 → ISMS-P 준수
    
- **스타트업**: Terraform 코드 배포 전 `trivy config` 실행 → 오픈된 보안 그룹 차단
    
- **오픈소스 프로젝트**: Pull Request마다 Trivy CI 실행 → 취약한 의존성 조기 탐지
    

---

## 7️⃣ 토론 과제

1. Trivy가 CSPM과 CWPP에 동시에 활용될 수 있는 이유는 무엇인가?
    
2. Trivy의 장점을 바탕으로 소규모 스타트업에서도 쉽게 도입 가능한 이유를 설명해 보라.
    
3. False Positive 문제가 잦다면 어떻게 관리하는 것이 바람직한가?
    

---

## ✅ 정리

- Trivy는 **컨테이너, IaC, 클라우드 계정까지 점검 가능한 단일 스캐너**
    
- 원리는 취약점 DB와 대상 매칭을 통한 탐지
    
- DevSecOps 파이프라인의 개발, 빌드, 배포, 운영 단계 모두에 활용 가능
    
- 장점: 가볍고 범용적 / 한계: 런타임 보호 부족, False Positive 존재
    

👉 다음 주(3주차 이론): **CWPP – 컨테이너 워크로드 보안 이론**

---

# 2주차: Trivy 설치 및 기본 사용법 (실습)

---

## 🎯 학습 목표

- Trivy 설치 및 동작 확인
    
- 컨테이너 이미지 보안 점검 실습
    
- GitHub Actions 연동 경험
    

---

## Trivy 소개

- 오픈소스 보안 스캐너
    
- 점검 대상
    
    - 컨테이너 이미지 (CWPP)
        
    - IaC 코드 (Terraform, Kubernetes 등)
        
    - AWS 계정 설정 (CSPM)
        
- 출력: CLI 표, JSON, Markdown, SARIF
    

---

## 설치 – macOS/Linux

```bash
brew install aquasecurity/trivy/trivy
```

👉 설치 후 확인

```bash
trivy --version
```

예상 출력:

```
Version: 0.50.0
Vulnerability DB: 2025-09-05
```

---

## 설치 – Ubuntu

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | \
  sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

---

## 컨테이너 이미지 스캔 – Python

```bash
docker pull python:3.9
trivy image python:3.9
```

예상 출력:

```
python:3.9 (debian 11.7)
Total: 12 (LOW: 3, MEDIUM: 5, HIGH: 3, CRITICAL: 1)
```

---

## 컨테이너 이미지 스캔 – Nginx

```bash
docker pull nginx:latest
trivy image nginx:latest
```

예상 출력:

```
nginx:1.25 (debian 11.7)
Total: 8 (LOW: 2, MEDIUM: 3, HIGH: 2, CRITICAL: 1)
```

---

## JSON 결과 저장

```bash
mkdir -p reports
trivy image -f json -o reports/python.json python:3.9
trivy image -f json -o reports/nginx.json nginx:latest
```

👉 `reports/` 폴더에 저장됨

---

## GitHub Actions 연동

`.github/workflows/trivy.yml`

```yaml
name: Trivy Security Scan
on: [push]

jobs:
  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Trivy image scan
        run: trivy image python:3.9
      - name: Save JSON Report
        run: trivy image -f json -o trivy.json python:3.9
      - uses: actions/upload-artifact@v3
        with:
          name: trivy-report
          path: trivy.json
```

---

## 팀별 과제

1. `python:3.9` vs `nginx:latest` 결과 비교
    
2. JSON에서 Critical 취약점 1개 찾아 설명
    
3. 팀 Repo에 보고서 업로드
    
    - 실행 로그 + 스크린샷 + 분석 내용
        

---

## ✅ 정리

- Trivy 설치 및 버전 확인 완료
    
- 컨테이너 이미지 보안 점검 실습 진행
    
- JSON 저장 & GitHub Actions 연동 성공
    

👉 다음 주(3주차 실습): **CWPP – 컨테이너 이미지 심화 점검**

---
