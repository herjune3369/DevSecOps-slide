
# 6주차: 이론 – Mini Project ①, CSPM + CWPP 통합 리포트의 의미

---

## 🎯 학습 목표

- CSPM과 CWPP를 통합 점검해야 하는 이유를 설명할 수 있다
    
- 보안 리포트가 DevSecOps에서 어떤 역할을 하는지 이해한다
    
- 자동화된 보안 리포트의 활용 가치를 파악한다
    

---
<!-- _class: medium -->

## 1️⃣ 왜 CSPM과 CWPP를 통합해야 하는가?

클라우드 보안은 **단일 관점**으로는 부족하다.

- **CSPM**은 클라우드 계정과 IaC 보안 설정을 점검
    
    - IAM 권한 남용
        
    - Security Group 과다 개방
        
    - S3 퍼블릭 접근 허용
        
- **CWPP**는 실행 중인 워크로드(컨테이너, VM) 자체를 점검
    
    - 이미지 내 취약한 라이브러리
        
    - root 권한 실행
        
    - 런타임 행위 모니터링
        

👉 둘을 결합해야 **설정 보안**과 **실행 보안**을 함께 보장할 수 있음.  
예:

- IaC에서는 보안 그룹을 닫아놨는데 → 배포된 컨테이너 이미지에 Critical 취약점 존재
    
- 계정 MFA는 적용됐지만 → S3 버킷이 Public Access 허용 상태
    

---
<!-- _class: medium -->

## 2️⃣ 통합 점검의 필요성

- **공격자는 설정 취약점과 이미지 취약점을 동시에 노린다**
    
    - S3 퍼블릭 접근 + 취약한 Nginx 이미지 → 데이터 유출
        
    - IAM 권한 남용 + 컨테이너 Escape 취약점 → 계정 탈취
        
- **감사 및 규제 준수 대응**
    
    - 금융권: ISMS-P, PCI-DSS, GDPR
        
    - 공공기관: NIST, KISA 가이드라인
        
    - 통합 리포트가 “증적 자료” 역할
        
- **보안팀·개발팀 간 협업 촉진**
    
    - CSPM → 인프라 담당자 관점의 취약점
        
    - CWPP → 개발자/앱 담당자 관점의 취약점
        
    - 하나의 보고서에 모아야 서로 소통 가능
        

---
<!-- _class: medium -->

## 3️⃣ 리포트 자동화의 의미

수작업 보고서의 한계:

- 사람이 로그를 모으고 문서로 정리 → 시간 소모
    
- 최신 상태 반영 어려움
    

자동화된 리포트의 장점:

1. **일관성**
    
    - 같은 형식으로 매번 결과 제공
        
    - 비교·추세 분석 가능
        
2. **가독성**
    
    - JSON → Markdown/PDF 변환
        
    - 관리자가 이해하기 쉬운 언어로 변환
        
3. **연속성**
    
    - GitHub Actions 등 CI/CD와 연결 → 코드 변경 시마다 최신 리포트 자동 생성
        

---

## 4️⃣ 리포트 구성 예시

```
# Security Scan Report

## Container Scan (CWPP)
- python:3.9 → Critical 1, High 3
- nginx:latest → Critical 1, High 2

## IaC Scan (CSPM)
- Security Group 0.0.0.0/0 허용 (HIGH)
- RDS 암호화 미적용 (MEDIUM)

## AWS Account Scan (CSPM)
- Root MFA 미사용 (HIGH)
- S3 Public Access 허용 (CRITICAL)

## 종합 분석
- Top 3 취약점:
  1. S3 Public Access (CRITICAL)
  2. Root MFA 미사용 (HIGH)
  3. Nginx 최신 이미지 Critical 취약점
```

👉 이 구조를 자동으로 뽑아낼 수 있어야 팀 내 공유와 경영진 보고가 쉬워짐.

---

## 5️⃣ 산업 적용 사례

- **금융기관**
    
    - 매주 CSPM + CWPP 리포트 생성 → 보안위원회에 보고
        
    - 감사 대응 시 **PDF 보고서**를 그대로 제출
        
- **스타트업**
    
    - GitHub Actions + Trivy 연동 → Pull Request마다 자동 보안 리포트 생성
        
    - CTO가 리포트를 확인하고 merge 승인 여부 결정
        
- **공공기관**
    
    - IaC 보안 가이드 준수 검증 결과 → CSPM 리포트로 제출
        
    - ISMS-P 심사 시 증적 자료로 활용
        

---

## 6️⃣ 토론 과제

1. CSPM과 CWPP를 따로 점검하는 것과 통합 리포트로 관리하는 것의 차이는 무엇인가?
    
2. DevSecOps 파이프라인에서 리포트 자동화가 중요한 이유는?
    
3. 여러분이 DevOps 엔지니어라면, **리포트 수신 대상**을 누구로 설정하겠는가? (개발자/보안팀/경영진)
    

---

## ✅ 정리

- CSPM은 설정 보안, CWPP는 워크로드 보안 → 둘 다 필요
    
- 통합 리포트는 보안팀·개발팀 협업, 규제 준수, 경영진 보고에 유용
    
- 자동화된 보고서는 최신 상태를 보장하고 증적 자료 역할을 한다
    

👉 다음 주(7주차 이론): **CI/CD 파이프라인과 보안 자동화 개념**

---
# 6주차: Mini Project ① – Trivy로 CSPM + CWPP 리포트 생성 (실습, 3시간)

---

## 🎯 학습 목표

- 컨테이너 이미지(CWPP)와 IaC/AWS 계정(CSPM)을 **종합 점검**한다
    
- Trivy 결과를 JSON/Markdown/PDF 리포트로 변환한다
    
- 팀별 분석·발표를 통해 **실무형 보안 리포트 작성**을 경험한다
    

---

## 1️⃣ 프로젝트 개요 (20분)

- 지금까지 배운 **CWPP + CSPM**을 하나로 합쳐 **리포트 프로젝트** 수행
    
- 대상
    
    - CWPP: 컨테이너 이미지 (`python:3.9`, `nginx:latest`)
        
    - CSPM: IaC 코드(`terraform/`), AWS 계정(`ap-northeast-2`)
        
- 최종 산출물
    
    - JSON 결과 (`reports/*.json`)
        
    - 통합 리포트 (`final_report.md`, `final_report.pdf`)
        
    - 팀 발표 자료
        

---

## 2️⃣ Trivy 종합 점검 실습 (60분)

### Step 1. 컨테이너 이미지 점검

```bash
docker pull python:3.9
docker pull nginx:latest

trivy image -f json -o reports/python.json python:3.9
trivy image -f json -o reports/nginx.json nginx:latest
```

👉 **확장 미션**: 팀별로 `mysql`, `redis`, `ubuntu:22.04` 추가 점검

---

### Step 2. IaC 코드 점검

```bash
trivy config -f json -o reports/terraform.json ./terraform
```

예상 발견 취약점:

- Security Group 0.0.0.0/0 허용
    
- RDS 암호화 미적용
    

👉 **에러 미션**: SG 코드에서 `cidr_blocks = ["0.0.0.0/0"]` 유지 → 취약점 발생 → 수정 후 재점검

---

### Step 3. AWS 계정 점검

```bash
trivy aws --region ap-northeast-2 -f json -o reports/aws.json
```

예상 발견 취약점:

- Root MFA 미사용
    
- S3 Public Access 허용
    

👉 **확장 미션**: `aws s3control put-public-access-block ...` 실행 후 다시 점검 → 차이 확인

---

## 3️⃣ 리포트 자동 생성 실습 (30분)

```bash
python generate_security_report.py \
  --inputs reports/python.json reports/nginx.json \
           reports/terraform.json reports/aws.json \
  --output reports/final_report.md
```

👉 결과:

- `final_report.md` (Markdown 리포트)
    
- `final_report.pdf` (PDF 리포트)
    

---

## 4️⃣ 팀별 분석 과제 (40분)

1. 팀별로 리포트 내용 정리
    
    - Critical/High 취약점 2개 선정
        
    - 위험 설명 + 개선 방법 작성
        
2. Markdown에 **Top 3 취약점 우선순위** 섹션 추가
    
3. 팀 Repo에 `reports/final_report.md` 업로드
    
4. 발표용 슬라이드 3장 이내 제작
    

---

## 5️⃣ 발표 & 토론 (30분)

- 각 팀 발표: **취약점 요약 → 영향 → 개선 방안**
    
- 다른 팀 결과 비교 → 공통 취약점, 특이 취약점 공유
    
- 토론: “현업에서는 이 리포트를 어떻게 활용할 수 있을까?”
    

---

## ✅ 정리

- CSPM + CWPP 종합 점검으로 클라우드 보안 리스크 확인
    
- Trivy 결과를 자동 변환해 실무 보고서 작성 경험
    
- 팀별 비교·분석을 통해 우선순위 기반 보안 개선 전략 수립
    

👉 다음 주(7주차 실습): **GitHub Actions에 Trivy 보안 스캔 연동**

---
