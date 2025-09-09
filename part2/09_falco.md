# 9주차: 이론 – 보안 리포트 자동화와 Markdown/PDF 변환

---

## 🎯 학습 목표

- 보안 리포트 자동화의 필요성을 이해한다
    
- JSON 결과를 Markdown/PDF로 변환하는 이유를 설명할 수 있다
    
- 이해관계자(개발자/보안팀/경영진)별 리포트 활용 방식을 구분할 수 있다
    

---

## 1️⃣ 왜 보안 리포트를 자동화해야 하는가?

- **수작업 보고의 한계**
    
    - 시간이 오래 걸림
        
    - 최신 상태 반영 불가
        
    - 담당자마다 형식이 달라 일관성 부족
        
- **자동화 리포트 장점**
    
    - 즉시성: 코드/배포 변경 시 자동 생성
        
    - 일관성: 동일한 포맷 유지
        
    - 추적성: 기록이 남아 추세 분석 가능
        

---

## 2️⃣ 리포트 자동화의 일반적 흐름

1. **스캔 도구 실행**
    
    - Trivy → JSON 결과 생성
        
2. **결과 변환**
    
    - Python 등 스크립트 → JSON → Markdown/PDF
        
3. **CI/CD 파이프라인 통합**
    
    - GitHub Actions, Jenkins 등에서 자동 실행
        
4. **결과 공유**
    
    - GitHub Repo 저장
        
    - 이메일/Slack/Teams 알림
        

---

## 3️⃣ JSON → Markdown/PDF 변환의 의미

- **JSON**
    
    - 기계가 읽기 좋은 데이터 구조
        
    - 자동화/통계 분석에 적합
        
- **Markdown**
    
    - 개발자 친화적
        
    - GitHub Repo에 그대로 반영 가능
        
- **PDF**
    
    - 경영진/감사 대응에 적합
        
    - 포맷이 고정되어 제출 자료로 활용
        

👉 동일한 데이터라도 **대상 독자별로 다른 포맷 제공** 필요

---

## 4️⃣ 리포트 구조 예시

```
# Security Scan Report

## Container Scan
- python:3.9 → Critical 1, High 3
- nginx:latest → Critical 1, High 2

## IaC Scan
- Security Group 0.0.0.0/0 허용 (HIGH)
- RDS 암호화 미적용 (MEDIUM)

## AWS Account Scan
- Root MFA 미사용 (HIGH)
- S3 Public Access 허용 (CRITICAL)

## 종합 분석
- Top 3 취약점
  1. S3 Public Access (CRITICAL)
  2. Root MFA 미사용 (HIGH)
  3. OpenSSL 취약점 (CRITICAL)
```

---

## 5️⃣ 이해관계자별 활용 방식

- **개발자**
    
    - 코드에 직접 반영할 취약점 정보 필요
        
    - CVE ID, 패키지명, 패치 버전
        
- **보안팀**
    
    - 위험도별 전체 현황, 우선순위 파악
        
    - Critical/High 중심
        
- **경영진**
    
    - “현재 리스크 수준”과 “규제 준수 상태”만 요약 필요
        
    - 세부 CVE보다는 **비즈니스 영향** 강조
        

---

## 6️⃣ 사례

- **스타트업**: GitHub Actions 실행 시 Markdown 리포트 자동 생성 → Repo에 기록
    
- **대기업**: 매일 새벽 PDF 리포트 생성 → 보안팀과 CTO 메일 발송
    
- **공공기관**: 월 단위 PDF 리포트 제출 → 규제 감사 대응
    

---

## 7️⃣ 토론 과제

1. JSON 결과만 있고 변환 리포트가 없다면 어떤 문제가 생길까?
    
2. 개발자와 경영진에게 동일한 리포트를 주면 비효율적인 이유는 무엇인가?
    
3. 우리 팀이라면 Markdown과 PDF 중 어떤 포맷을 더 우선시해야 할까?
    

---

## ✅ 정리

- 보안 리포트 자동화는 DevSecOps 파이프라인의 핵심 요소
    
- JSON은 기계용, Markdown은 개발자용, PDF는 경영진·규제 대응용
    
- 이해관계자별 요구에 맞춘 리포트 제공이 중요하다
    

👉 다음 주(10주차 이론): **SARIF 표준과 GitHub Security Tab 활용**

---
# 9주차: Trivy 결과 → Markdown/PDF 리포트 자동 생성 (실습, 3시간)

---

## 🎯 학습 목표

- Trivy JSON 결과를 수집하고 변환한다
    
- Python 스크립트를 활용해 Markdown/PDF 리포트를 자동 생성한다
    
- 팀별로 리포트를 분석하여 취약점 개선안을 도출한다
    

---

## 1️⃣ 사전 준비 (20분)

### 실습 폴더 생성

```bash
mkdir -p reports
```

### 기존 스캔 결과 확보

```bash
trivy image -f json -o reports/python.json python:3.9
trivy config -f json -o reports/terraform.json ./terraform
trivy aws --region ap-northeast-2 -f json -o reports/aws.json
```

👉 `reports/` 폴더에 JSON 결과 3개 저장됨

---

## 2️⃣ Python 변환 스크립트 실행 (40분)

`generate_security_report.py`

```bash
python generate_security_report.py \
  --inputs reports/python.json reports/terraform.json reports/aws.json \
  --output reports/final_report.md
```

👉 결과 파일

- `final_report.md`
    
- `final_report.pdf`
    

---

## 3️⃣ 결과 리포트 구조 확인 (30분)

Markdown(`final_report.md`) 예시:

```
# Security Scan Report

## Container Scan
- python:3.9 → Critical 1, High 3

## IaC Scan
- SG 0.0.0.0/0 허용 (HIGH)

## AWS Account Scan
- Root MFA 미사용 (HIGH)
- S3 Public Access 허용 (CRITICAL)

## 종합 분석
- Top 3 취약점:
  1. S3 Public Access (CRITICAL)
  2. Root MFA 미사용 (HIGH)
  3. OpenSSL 취약점 (CRITICAL)
```

---

## 4️⃣ 확장 미션 (40분)

1. JSON 3종 중 한 개를 직접 수정 → 잘못된 값 입력 → 스크립트 오류 확인
    
2. Markdown에서 **Critical/High 취약점만** 필터링 기능 추가
    
3. PDF 출력 후 `scan-report/sample.pdf` 와 비교
    

---

## 5️⃣ 팀별 과제 (30분)

- 각 팀은 생성된 `final_report.md`에서 **Top 3 취약점** 정리
    
- 개선 방안(코드 수정, AWS CLI 명령 등) 제시
    
- Repo에 `reports/team_report.md` 업로드
    

---

## 6️⃣ 발표 & 토론 (20분)

- 팀별 발표 (5분)
    
- 비교: 어떤 팀이 더 심각한 취약점을 발견했는가?
    
- 토론: **리포트 자동화가 DevSecOps 파이프라인에서 중요한 이유**
    

---

## ✅ 정리

- Trivy JSON 결과를 Python 스크립트로 Markdown/PDF 리포트 자동 변환 성공
    
- 자동 리포트를 통해 취약점 현황을 쉽게 파악 가능
    
- 팀별 분석을 통해 취약점 대응 방안 학습
    

👉 다음 주(10주차 실습): **Trivy 결과 → SARIF 변환 및 GitHub Security Tab 업로드**

---
