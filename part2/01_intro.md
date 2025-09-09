
# 📘 클라우드 보안 및 DevSecOps

## Part 1  – DevSecOps 기초 & Trivy (CSPM + CWPP) 
## Part 2 – Ansible 자동화 & DevSecOps 


---

## 🎯 Part 1 학습 목표 – DevSecOps 기초 & Trivy (CSPM + CWPP) 

- DevOps와 DevSecOps 차이 및 **Shift Left Security** 이해
    
- **CSPM, CWPP, CNAPP** 보안 프레임워크 개념 습득
    
- Trivy를 활용해 **컨테이너, IaC, AWS 계정 보안 점검** 수행
    
- JSON/Markdown 리포트 해석 및 **취약점 개선 방안 도출**
    
- CSPM + CWPP 결과를 팀별 **보안 리포트로 정리·발표**

---
## Part 1.  강의 목차

- **1주차**: DevSecOps 개요 & 프레임워크 소개
    
- **2주차**: Trivy 설치 및 기본 사용법
    
- **3주차**: CWPP – 컨테이너 이미지 취약점 점검
    
- **4주차**: CSPM – IaC 보안 점검
    
- **5주차**: CSPM – AWS 계정 보안 점검
    
- **6주차**: Mini Project ① – CSPM + CWPP 종합 리포트

---


## 🎯 Part 2 학습 목표 – Ansible 자동화 & DevSecOps 

- Ansible로 **애플리케이션 및 DB 자동 배포** 수행
    
- GitHub Actions로 **CI/CD 파이프라인** 구성 및 Terraform+Ansible 연동
    
- Trivy를 CI/CD에 통합해 **CSPM + CWPP 보안 점검 자동화**
    
- 보안 리포트를 **Markdown/PDF/SARIF**로 변환해 관리·공유
    
- 팀별 프로젝트를 통해 **DevSecOps 파이프라인 완성**
    

---
## Part 2.  강의목차

- **7주차**: GitHub Actions에 Trivy 보안 스캔 연동
    
- **8주차**: `.trivyignore` 활용 및 False Positive 관리
    
- **9주차**: Trivy 결과 → Markdown/PDF 리포트 자동 생성
    
- **10주차**: Trivy 결과 → SARIF 변환 및 GitHub Security Tab 업로드
    
- **11주차**: Mini Project ② – DevSecOps 파이프라인 완성
    
- **12주차**: 팀별 보안 자동화 프로젝트 기획
    
- **13주차**: 프로젝트 구현 및 테스트
    
- **14주차**: 최종 보고서 및 산출물 문서화
    
- **15주차**: 최종 발표 & GitHub 포트폴리오 정리
---

#  Part 1  – DevSecOps 기초 & Trivy (CSPM + CWPP) 
---
# 1주차: DevSecOps 이론 – 개요와 필요성



## 🎯 학습 목표

- DevOps와 DevSecOps의 차이를 설명할 수 있다
    
- CSPM, CWPP, CNAPP 개념을 이해한다
    
- DevSecOps가 필요한 이유와 산업 적용 사례를 파악한다
    

---

## 1️⃣ DevOps → DevSecOps 진화

### DevOps

- **Dev(개발) + Ops(운영)**
    
- 자동화, 협업, 빠른 배포 → 효율성 강화
    
- CI/CD, IaC, 모니터링 중점
    

### DevSecOps

- DevOps 파이프라인 전 과정에 **보안(Security)** 내재화
    
- 코드 작성 → 빌드 → 배포 → 운영 단계마다 보안 점검
    
- **Shift Left Security**: 초기에 보안 문제 탐지·해결
    

👉 단순 “빠른 배포”에서 → “안전한 빠른 배포”로 진화

---

## 2️⃣ DevSecOps가 필요한 이유

- **클라우드 보편화**: IaC, 컨테이너, 마이크로서비스 확산
    
- **공격 표면 확대**: 잘못된 설정, 취약한 이미지, 오픈된 계정
    
- **규제 준수 요구**: 금융, 공공, 의료 등 산업별 보안 인증 필요
    
- **실제 사고 사례**
    
    - AWS S3 Public Access → 대규모 개인정보 유출
        
    - 컨테이너 이미지 취약점 방치 → 랜섬웨어 감염
        

---

## 3️⃣ 보안 프레임워크

- **CSPM (Cloud Security Posture Management)**
    
    - IaC, 클라우드 계정 설정 점검
        
    - 잘못된 보안 그룹, 공개 버킷, MFA 미사용 등 탐지
        
- **CWPP (Cloud Workload Protection Platform)**
    
    - 컨테이너, VM, 서버리스 워크로드 점검
        
    - 취약점 스캔, 런타임 보호, 규정 준수
        
- **CNAPP (Cloud-Native Application Protection Platform)**
    
    - CSPM + CWPP 통합 관리
        
    - 클라우드 보안의 차세대 표준
        

---

## 4️⃣ DevSecOps 구현 흐름

1. **IaC 보안**
    
    - Terraform, Ansible 코드 점검
        
    - CSPM 활용 (`trivy config`, `trivy aws`)
        
2. **워크로드 보안**
    
    - 컨테이너 이미지 점검
        
    - CWPP 활용 (`trivy image`)
        
3. **CI/CD 파이프라인 보안**
    
    - GitHub Actions, Jenkins 등 자동화
        
    - 보안 스캔 결과를 빌드/배포 전 단계에서 반영
        

---

## 5️⃣ 산업 적용 사례

- **금융권**
    
    - CSPM: AWS 계정 정책 준수 여부 자동화 점검
        
    - CWPP: Kubernetes Pod 이미지 스캔
        
    - 규제 대응 (ISMS-P, PCI-DSS)
        
- **스타트업**
    
    - GitHub Actions + Trivy 연동
        
    - 배포 전 이미지 스캔, IaC 코드 보안 점검 자동화
        
- **공공기관**
    
    - IaC 점검(CSPM)으로 오픈된 보안 그룹 차단
        
    - 취약점 리포트 자동화해 감사 대응
        

---

## 6️⃣ 토론 과제

1. DevOps와 DevSecOps의 가장 큰 차이는 무엇인가?
    
2. CSPM과 CWPP는 서로 어떤 관계를 맺고 있는가?
    
3. 우리 조직(가상의 팀)에 적용한다면, **DevSecOps의 가장 우선 과제**는 무엇일까?
    

---

## ✅ 정리

- DevSecOps는 DevOps의 빠른 배포에 보안을 내재화한 진화된 개념
    
- CSPM, CWPP, CNAPP은 클라우드 보안의 핵심 프레임워크
    
- IaC, 워크로드, CI/CD 파이프라인 단계에서 모두 보안 점검 필요
    
- 실제 산업 사례를 통해 DevSecOps의 필요성을 확인했다
    

👉 다음 주(2주차 이론): **Trivy – 오픈소스 보안 스캐너의 역할과 원리**

---

# 1주차: DevSecOps 개요 & 실습 환경 준비 (3시간)

---

## 🎯 학습 목표

- DevOps와 DevSecOps 차이 이해
    
- CSPM, CWPP, CNAPP 프레임워크 설명 가능
    
- AWS, Terraform, Ansible, Trivy 실습 환경 준비
    
- EC2 생성, Ansible 연결, Trivy 첫 점검 수행
    

---

## 1️⃣ DevOps → DevSecOps 개요 (30분)

- DevOps: 개발 + 운영 자동화 (CI/CD)
    
- DevSecOps: DevOps 파이프라인 전 과정에 **보안(Security)** 내재화
    
- Shift Left Security: 개발 단계에서 보안 강화
    

### 프레임워크 소개

- CSPM: IaC, 클라우드 설정 점검
    
- CWPP: 워크로드(이미지, VM) 점검
    
- CNAPP: CSPM + CWPP 통합
    

---

## 2️⃣ 환경 준비 (60분)

### AWS CLI 설정

```bash
aws configure
```

👉 Access Key, Secret Key, Region(`ap-northeast-2`) 입력

### Terraform 설치 확인

```bash
terraform -version
```

👉 예상 출력: `Terraform v1.5.x`

---
### Ansible 설치 확인

```bash
ansible --version
```

👉 예상 출력: `ansible [core 2.15.x]`

### Trivy 설치 확인

```bash
trivy --version
```

👉 예상 출력

```
Version: 0.50.0
Vulnerability DB: 2025-09-05
```

👉 **과제**: 설치 로그 + 버전 출력 캡처 → Repo 업로드

---

## 3️⃣ 첫 실습 – 코드 구조 확인 & EC2 배포 (60분)

### GitHub Repo 클론

```bash
git clone https://github.com/your-org/devsecops-sample.git
cd devsecops-sample
ls -R
```

👉 주요 구조

```
terraform/
ansible/
.github/workflows/
trivy-report/
```
---
### Terraform으로 EC2 생성

```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

👉 출력:

```
Apply complete! Resources: 1 added.
Outputs:
ec2_public_ip = "3.35.xxx.xxx"
```

👉 **확장 미션**: `variables.tf` 수정 → 인스턴스 타입 `t2.micro → t3.micro` 변경 후 재배포

---

### Ansible ping 테스트

`ansible/inventories/hosts.ini`

```
[app]
3.35.xxx.xxx
```

```bash
ansible -i inventories/hosts.ini app -m ping
```

👉 예상 출력

```
3.35.xxx.xxx | SUCCESS => {
  "changed": false,
  "ping": "pong"
}
```

👉 **확장 미션**: `ansible/playbook.yml`에 `apt update` Task 추가 후 실행

---

## 4️⃣ 보안 점검 맛보기 (30분)

### 컨테이너 이미지 점검

```bash
trivy image ubuntu:20.04
```

👉 출력

```
Total: 5 (LOW: 1, MEDIUM: 2, HIGH: 2, CRITICAL: 0)
```

---
### IaC 코드 점검

```bash
trivy config ./terraform
```

👉 출력

```
aws_security_group.my_sg
  └ Severity: HIGH
  └ Security group allows ingress from 0.0.0.0/0 to port 22
```

👉 **확장 미션**: SG 코드를 수정 → 특정 IP만 허용 후 다시 점검

---

## 5️⃣ 팀별 과제 (30분)

1. EC2 생성 & Ansible ping 결과 캡처
    
2. `trivy image ubuntu:20.04` 실행 결과 캡처
    
3. 팀 Repo에 Markdown 보고서 작성
    
    - 실행 로그 + 스크린샷 + 배운 점 3가지
        

---

## ✅ 오늘의 정리

- DevOps와 DevSecOps 차이, CSPM/CWPP/CNAPP 개념 이해
    
- Terraform/Ansible/Trivy 설치 및 기본 동작 확인
    
- EC2 배포, Ansible 연결, Trivy 첫 점검 완료
    
- 보안 그룹 규칙과 취약점 점검의 중요성 체험
    

👉 다음 주(2주차 실습): **Trivy 설치 및 기본 사용법 심화**

---
