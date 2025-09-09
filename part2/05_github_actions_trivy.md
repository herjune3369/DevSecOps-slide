# 5주차: CSPM 이론 – AWS 계정 보안 점검

---

## 🎯 학습 목표

- CSPM의 **계정 단위 점검** 개념 이해
    
- AWS 계정에서 발생하는 보안 취약점 유형 파악
    
- CSPM 도구를 활용한 계정 보안 관리 필요성 인식
    

---

## 1️⃣ AWS 계정 보안의 중요성

- AWS 계정은 **클라우드 리소스 전체 접근 권한**을 가짐
    
- 잘못된 계정 설정은 인프라 전체 보안사고로 직결
    
- 예:
    
    - Root 계정 MFA 미사용 → 계정 탈취 위험
        
    - S3 Public Access 허용 → 데이터 유출
        
    - 과도한 IAM 권한 → 내부자 위협
        

---

<!-- _class: medium -->

## 2️⃣ CSPM at Account Level

- CSPM은 단순 IaC 코드 점검을 넘어 **클라우드 계정 설정 자체**를 점검
    
- 주요 점검 범위
    
    1. **IAM (Identity & Access Management)**
        
        - Root 계정 MFA
            
        - 과도한 권한 정책 (AdministratorAccess 남용)
            
    2. **S3**
        
        - Public Access 허용 여부
            
        - 기본 암호화 설정
            
    3. **RDS**
        
        - Storage 암호화 여부
            
        - Public 접근 차단 여부
            
    4. **네트워크 (VPC/SG)**
        
        - 보안 그룹 0.0.0.0/0 오픈
            
    5. **로깅/모니터링**
        
        - CloudTrail 비활성화
            
        - 로그 암호화 누락
            

---

## 3️⃣ Trivy와 AWS 계정 점검

- `trivy aws --region ap-northeast-2`
    
- 계정 설정 전반을 스캔 → JSON, Table 형식 결과 출력
    
- 발견된 취약점: CVE가 아닌 **Best Practice 위반** 중심
    

---

## 4️⃣ CSPM 도구 비교

- **Trivy**: 가볍고 빠름, DevSecOps 파이프라인에 적합
    
- **ScoutSuite**: 멀티 클라우드 계정 점검 지원
    
- **CloudSploit (Aqua Security)**: 규정 준수 기반 점검
    
- **상용 솔루션 (Prisma Cloud, Wiz, Orca Security)**
    
    - CSPM + CWPP 통합(CNAPP) 기능 제공
        
    - 대규모 조직, 규제 대응 목적
        

---

## 5️⃣ 적용 사례

- **금융권**: Root 계정 MFA, CloudTrail 활성화 → ISMS-P 필수
    
- **스타트업**: S3 Public Access Block → 데이터 유출 방지
    
- **공공기관**: 계정 정책 위반 자동 리포트 생성 → 감사 대응
    

---

## 6️⃣ 토론 과제

1. AWS 계정 보안에서 가장 치명적인 설정 오류는 무엇인가?
    
2. Root 계정 사용을 최소화해야 하는 이유는?
    
3. CSPM 계정 점검을 자동화해야 하는 이유는 무엇인가?
    

---

## ✅ 정리

- CSPM은 AWS 계정 설정 수준에서 보안 상태를 점검
    
- IAM, S3, RDS, 네트워크, 로깅이 주요 점검 대상
    
- Trivy는 계정 설정을 자동 점검하여 Best Practice 위반을 탐지
    
- CSPM 계정 점검은 규제 준수와 실제 보안사고 예방에 필수
---
# 5주차: CSPM – AWS 계정 설정 점검 (실습, 3시간)

---

## 🎯 학습 목표

- CSPM의 계정 단위 점검 수행
    
- Trivy로 IAM, S3, RDS, VPC 보안 설정 점검
    
- 점검 결과 해석 및 개선 방안 적용
    

---

## 1️⃣ AWS 계정 점검 개요 (20분)

- CSPM은 **계정 단위 보안 설정 점검**을 수행
    
- 주요 점검 대상
    
    - IAM: Root MFA, 과도 권한 정책
        
    - S3: Public Access, 암호화 여부
        
    - RDS: 암호화 및 접근 제어
        
    - VPC: 보안 그룹 0.0.0.0/0 오픈
        
    - CloudTrail: 로깅 활성화
        

---

## 2️⃣ 사전 준비 (20분)

### AWS CLI 인증

```bash
aws configure
```

👉 Access Key, Secret Key, Region(`ap-northeast-2`) 입력

### Trivy 설치 확인

```bash
trivy --version
```

👉 출력 예시: `Version: 0.50.0`

---
<!-- _class: medium -->

## 3️⃣ Trivy AWS 계정 점검 (40분)

### 기본 점검

```bash
trivy aws --region ap-northeast-2
```

예상 출력:

```
IAMUser/RootUser
  └ MFA not enabled   (Severity: HIGH)

S3Bucket/my-bucket
  └ Public Access Block disabled   (Severity: CRITICAL)

RDSInstance/mydb
  └ Storage encryption not enabled   (Severity: MEDIUM)
```

### JSON 저장

```bash
mkdir -p reports
trivy aws --region ap-northeast-2 -f json -o reports/aws.json
```

---
<!-- _class: medium -->

## 4️⃣ 개선 실습 (40분)

### Root 계정 MFA 활성화

- AWS 콘솔 → IAM → Root 계정 → MFA 활성화
    

### S3 Public Access 차단

```bash
aws s3api put-bucket-acl --bucket my-bucket --acl private

aws s3control put-public-access-block \
  --account-id <ACCOUNT_ID> \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### RDS 암호화 적용

- Terraform 코드에서 `storage_encrypted = true` 추가
    
- 새 DB 인스턴스로 배포 후 점검
    

---

## 5️⃣ 팀별 실습 과제 (30분)

1. `trivy aws` 실행 후 JSON 결과 저장
    
2. Critical/High 취약점 2개 선택
    
3. 개선 방법 적용 → 재점검 결과 비교
    
4. 팀 Repo에 보고서 업로드
    
    - 실행 로그 + 스크린샷 + 개선 코드/CLI 명령
        

---

## 6️⃣ 발표 & 토론 (30분)

- 각 팀 발표: 취약점 2개, 개선 방법, 재점검 결과
    
- 공통적으로 발생한 문제와 해결 방식 비교
    
- 토론 주제: **“운영 환경에서 계정 보안 자동화를 어떻게 구현할 것인가?”**
    

---

## ✅ 정리

- CSPM은 계정 단위에서 IAM, S3, RDS, VPC 설정을 점검
    
- Trivy `trivy aws`로 보안 설정 자동 점검 가능
    
- Root MFA, S3 Public Access, RDS 암호화가 핵심 이슈
    
- 개선 후 재점검으로 보안 향상 효과 확인
---
