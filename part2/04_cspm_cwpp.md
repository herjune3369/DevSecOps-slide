# 4주차: CSPM 이론 – IaC 보안 개념과 필요성

---

## 🎯 학습 목표

- CSPM 개념과 역할 설명
    
- IaC 보안 점검 필요성 이해
    
- IaC에서 발생하는 주요 보안 취약점 파악
    

---

## 1️⃣ CSPM 개요

- **CSPM (Cloud Security Posture Management)**
    
    - 클라우드 인프라 및 계정 설정의 보안 상태 점검
        
    - 잘못된 구성(Misconfiguration) 탐지 및 개선
        
- **DevSecOps에서의 역할**
    
    - IaC 코드 보안 → 사전 점검
        
    - 운영 계정 보안 → 사후 점검
        

---

## 2️⃣ IaC 보안의 필요성

- **IaC (Infrastructure as Code)**: 인프라를 코드로 정의 (Terraform, CloudFormation 등)
    
- 장점: 자동화, 재현 가능, 관리 용이
    
- 위험: 잘못된 코드 배포 → 보안 문제 확산
    
- 예:
    
    - Security Group 0.0.0.0/0 → 누구나 접근 가능
        
    - RDS 암호화 옵션 누락 → 민감 데이터 노출
        
    - 하드코딩된 비밀번호
        

---
<!-- _class: medium -->
## 3️⃣ CSPM 주요 점검 항목

1. **네트워크 보안**
    
    - Security Group 과도 개방 여부
        
    - VPC/Subnet 구성 오류
        
2. **데이터 보안**
    
    - 암호화 미적용 (RDS, S3)
        
    - Public Access 허용
        
3. **계정 및 권한**
    
    - Root 계정 MFA 미사용
        
    - IAM 정책 과도 권한
        
4. **로깅 및 모니터링**
    
    - CloudTrail 비활성화
        
    - 로그 암호화 누락
        

---

## 4️⃣ CSPM 도구와 방식

- **Trivy**: `trivy config`, `trivy aws`
    
- **Checkov**: IaC 코드 점검 전문
    
- **Cloud Custodian**: 정책 기반 클라우드 관리
    
- **상용 솔루션**: Prisma Cloud, Wiz, Orca Security 등
    

---

## 5️⃣ CSPM 적용 사례

- **금융기관**: Terraform 코드 점검으로 ISMS-P 인증 대응
    
- **스타트업**: S3 Public Access 탐지 자동화 → 데이터 유출 예방
    
- **공공기관**: IaC 코드 + 계정 정책 점검 → 보안 감사 대응
    

---

## 6️⃣ 토론 과제

1. IaC 보안 점검을 하지 않으면 어떤 문제가 발생할까?
    
2. CSPM 점검 항목 중 실제 운영 환경에서 가장 중요한 것은 무엇인가?
    
3. CSPM과 CWPP가 통합된 CNAPP의 장점은 무엇인가?
    

---

## ✅ 정리

- CSPM은 클라우드 인프라 및 계정의 보안 구성을 점검하는 체계
    
- IaC 보안 점검은 잘못된 코드 배포로 인한 보안 사고를 예방
    
- 주요 점검 포인트: 네트워크, 데이터, 계정, 로깅
    
- CSPM 도구와 상용 솔루션을 활용해 자동화 가능
---
# 4주차: CSPM – IaC 보안 점검 (실습, 3시간)

---

## 🎯 학습 목표

- CSPM 개념 이해
    
- Trivy로 Terraform 코드 보안 점검 수행
    
- IaC 보안 이슈 해석 및 코드 수정으로 개선
    

---

## 1️⃣ IaC 코드 구조 확인 (30분)

```bash
cd terraform
ls
```

예상 구조

```
main.tf
variables.tf
outputs.tf
```

- **main.tf**: AWS 리소스 정의 (VPC, EC2, SG 등)
    
- **variables.tf**: 변수 정의
    
- **outputs.tf**: 출력 값
    

---

## 2️⃣ Trivy IaC 보안 점검 (40분)

```bash
trivy config ./terraform
```

예상 출력

```
aws_security_group.my_sg
  └ Severity: HIGH
  └ Security group allows ingress from 0.0.0.0/0 to port 22

aws_db_instance.mydb
  └ Severity: MEDIUM
  └ Storage encryption is not enabled
```

JSON 저장

```bash
trivy config -f json -o reports/terraform.json ./terraform
```

---

## 3️⃣ 코드 수정 & 재점검 (40분)

### 보안 그룹 개선

```hcl
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["203.0.113.25/32"] # 특정 IP만 허용
}
```

### RDS 암호화 적용

```hcl
storage_encrypted = true
```

수정 후 다시 실행

```bash
trivy config ./terraform
```

👉 개선된 결과 비교

---

## 4️⃣ 팀별 실습 과제 (30분)

1. 팀별로 `trivy config ./terraform` 실행 결과 캡처
    
2. Critical/High 취약점 2개 이상 개선 코드 작성
    
3. 수정 전/후 결과 비교표 작성
    
4. 팀 Repo에 Markdown 보고서 업로드
    

---

## 5️⃣ 발표 & 토론 (30분)

- 각 팀 취약점 및 개선 방법 발표 (5분)
    
- 공통적으로 발견된 취약점 논의
    
- 토론: “IaC 코드에서 가장 먼저 점검해야 할 보안 포인트는 무엇인가?”
    

---

## ✅ 정리

- CSPM은 IaC 코드의 보안 상태를 점검하는 체계
    
- Trivy `trivy config`로 Terraform 코드 점검 가능
    
- SG 과다 개방, 암호화 미적용 등 주요 취약점 식별
    
- 코드 수정 후 재점검으로 개선 효과 확인
