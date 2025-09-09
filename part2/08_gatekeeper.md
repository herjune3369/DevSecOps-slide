
# 8주차: 이론 – False Positive와 보안 리포트 신뢰성 관리

---

## 🎯 학습 목표

- False Positive(거짓 양성)의 개념을 이해한다
    
- 보안 점검 결과에서 False Positive가 발생하는 원인을 설명할 수 있다
    
- DevSecOps 파이프라인에서 False Positive를 관리하는 방법을 학습한다
    

---

## 1️⃣ False Positive란?

- **False Positive**: 실제로는 안전한 항목을 보안 도구가 취약점으로 잘못 탐지하는 것
    
- 반대 개념 → **False Negative**: 실제 취약점이 있음에도 불구하고 탐지하지 못하는 것
    

---

## 2️⃣ False Positive가 발생하는 이유

1. **취약점 DB와 버전 불일치**
    
    - 소프트웨어 버전은 최신이지만 DB에 오래된 정보가 남아있는 경우
        
2. **패치 적용 오인식**
    
    - 패치가 이미 적용된 상태인데 도구가 탐지하지 못하고 여전히 취약하다고 판단
        
3. **환경 차이**
    
    - 운영 환경에서 실제로는 사용하지 않는 라이브러리를 탐지 대상에 포함
        
4. **도구 한계**
    
    - 단순 문자열 매칭 방식 → 컨텍스트 반영 부족
        

---

## 3️⃣ False Positive의 문제점

- **경고 피로(Alert Fatigue)**: 너무 많은 경고로 인해 진짜 중요한 취약점을 놓침
    
- **개발자 생산성 저하**: 의미 없는 이슈 해결에 시간 낭비
    
- **보안팀과 개발팀 갈등**: “실제 취약점 아님” vs “도구 결과 무시 불가”
    

---


## 4️⃣ DevSecOps에서 False Positive 관리 전략

### 1. `.trivyignore` 활용

- 특정 CVE ID를 무시하도록 설정
    
- 예시:
    
    ```
    CVE-2023-12345
    CVE-2022-54321
    ```
    

### 2. 결과 분류 체계화

- **Critical/High** 중심으로 우선 대응
    
- Medium/Low는 주기적 점검 또는 무시 처리
    
---
### 3. 주석 및 문서화

- 무시하는 취약점은 반드시 **사유 기록**
    
- “이미 패치 적용됨”, “운영 환경에서 미사용” 등 근거 남김
    

### 4. CI/CD 파이프라인 정책화

- 예: Critical 이상 취약점 발견 시 빌드 차단
    
- Medium 이하 발견 시 경고만 표시
    

---

## 5️⃣ 산업 적용 사례

- **스타트업**
    
    - `.trivyignore`로 반복 False Positive 관리
        
    - Critical 취약점만 배포 차단
        
- **대기업**
    
    - CI/CD 보안 게이트 → **Critical/High** 자동 차단
        
    - Medium/Low는 보안팀에서 주간 리포트로 관리
        
- **금융권**
    
    - 모든 취약점 보고 의무화
        
    - 단, False Positive는 “보안팀 검증 후 공식 예외 승인”
        

---

## 6️⃣ 토론 과제

1. False Positive가 너무 많을 때 개발팀은 어떤 반응을 보일까?
    
2. `.trivyignore`를 무분별하게 쓰면 어떤 위험이 있을까?
    
3. 우리 팀의 CI/CD에서는 어떤 기준으로 취약점을 차단해야 할까?
    

---

## ✅ 정리

- False Positive는 **보안 도구가 잘못 탐지한 취약점**
    
- 원인은 DB 불일치, 패치 오인식, 환경 차이, 도구 한계 등
    
- 관리 방법: `.trivyignore`, 우선순위 분류, 문서화, 파이프라인 정책화
    
- DevSecOps에서는 False Positive 관리가 곧 **보안 리포트 신뢰성 확보**로 이어진다
    

👉 다음 주(9주차 이론): **보안 리포트 자동화와 경영진 보고 관점**

---

# 8주차: `.trivyignore` 활용 및 False Positive 관리 (실습, 3시간)

---

## 🎯 학습 목표

- False Positive의 개념과 발생 원인 이해
    
- `.trivyignore` 파일을 활용해 반복 탐지되는 취약점 무시
    
- 팀별 정책 기반으로 취약점 관리 전략 수립
    

---

## 1️⃣ False Positive 확인 (30분)

### 기존 스캔 실행

```bash
trivy image python:3.9
```

👉 결과 일부

```
CVE-2023-12345 (HIGH)
Package: openssl
Fixed Version: 1.1.1t
Note: 이미 패치된 버전이지만 여전히 탐지됨
```

→ 이처럼 **실제 취약하지 않음에도 탐지되는 경우** = False Positive

---

## 2️⃣ `.trivyignore` 작성 (30분)

### 파일 생성

프로젝트 루트(`./`)에 `.trivyignore` 작성

```
CVE-2023-12345
CVE-2022-54321
```

### 재실행

```bash
trivy image python:3.9
```

👉 지정한 CVE는 결과에서 제외됨

---

## 3️⃣ IaC 및 AWS 점검에 적용 (40분)

### IaC 코드 점검

```bash
trivy config ./terraform
```

예상 결과

```
CVE-2023-22222  (LOW)
Hardcoded password found
```

→ 무시 대상에 추가

```
CVE-2023-22222
```
---
### AWS 계정 점검

```bash
trivy aws --region ap-northeast-2
```

예상 결과

```
IAMUser/RootUser: MFA not enabled (HIGH)
```

👉 MFA 미사용은 False Positive 아님 → 무시 금지

---

## 4️⃣ GitHub Actions에 `.trivyignore` 적용 (40분)

`.github/workflows/trivy.yml`

```yaml
- name: Run Trivy with ignore file
  run: trivy image --ignorefile .trivyignore -f json -o trivy.json python:3.9
```

👉 CI/CD 파이프라인에서도 자동 적용

---

## 5️⃣ 팀별 실습 과제 (20분)

1. 각 팀은 **3개 취약점 이상** 선택해 `.trivyignore` 적용
    
2. 적용 전/후 결과 비교 스크린샷 준비
    
3. “무시해도 되는 취약점” vs “절대 무시하면 안 되는 취약점” 분류
    

---

## 6️⃣ 발표 & 토론 (20분)

- 팀별 발표: `.trivyignore` 적용 결과 공유
    
- 토론:
    
    - 무분별한 무시는 왜 위험한가?
        
    - 우리 조직은 어떤 기준으로 False Positive를 관리해야 하는가?
        

---

## ✅ 정리

- False Positive는 **실제 취약하지 않음에도 탐지되는 항목**
    
- `.trivyignore`로 반복되는 항목을 관리 가능
    
- 단, 무시할 취약점과 반드시 대응할 취약점을 구분하는 기준이 필요
    

👉 다음 주(9주차 실습): **Trivy 결과 → Markdown/PDF 리포트 자동 생성**

