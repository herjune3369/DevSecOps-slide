
# 10주차: 이론 – SARIF 표준과 GitHub Security Tab 활용

---

## 🎯 학습 목표

- SARIF 포맷의 개념과 필요성을 이해한다
    
- Trivy 보안 결과를 SARIF로 변환해야 하는 이유를 설명할 수 있다
    
- GitHub Security Tab과 SARIF의 연계를 이해한다
    

---

## 1️⃣ SARIF 개요

- **SARIF (Static Analysis Results Interchange Format)**
    
    - 정적 분석 및 보안 스캐너 결과 교환을 위한 국제 표준(JSON 기반)
        
    - OASIS 표준으로 정의됨
        
- **등장 배경**
    
    - 보안/품질 도구마다 결과 포맷이 달라 통합 관리 어려움
        
    - 공통 포맷이 필요 → SARIF 채택
        

---

## 2️⃣ SARIF의 특징

1. **표준화**
    
    - CVE ID, 심각도, 영향 범위, 코드 위치를 통일된 구조로 표현
        
2. **호환성**
    
    - GitHub, Azure DevOps, VS Code 등 다양한 플랫폼에서 사용
        
3. **자동화 친화성**
    
    - CI/CD 파이프라인에 쉽게 통합 가능
        
4. **시각화 지원**
    
    - IDE, Repo Security Tab에서 취약점 바로 확인 가능
        

---

## 3️⃣ Trivy와 SARIF

- Trivy는 기본적으로 Table/JSON/Markdown 출력 지원
    
- `--format sarif` 옵션으로 SARIF 변환 가능
    
- SARIF 결과는 **GitHub Security Tab**에 업로드해 코드 스캐닝 결과로 반영
    

---

## 4️⃣ GitHub Security Tab의 역할

- **코드 스캐닝(Code Scanning Alerts)**
    
    - SARIF 업로드 시 자동으로 취약점 표시
        
- **협업 강화**
    
    - 개발자는 Pull Request에서 보안 이슈 즉시 확인 가능
        
- **규제/감사 대응**
    
    - 보안 결과를 Repo 단위로 중앙화해 관리
        

---

## 5️⃣ 적용 시나리오

- **개발자 관점**
    
    - 코드 변경 시 즉시 보안 피드백 제공
        
- **보안팀 관점**
    
    - 프로젝트별 취약점 현황을 한눈에 파악
        
- **경영진/감사 대응 관점**
    
    - 보안 상태를 Repo 기반으로 기록 및 증적 제출
        

---

## 6️⃣ 토론 과제

1. SARIF가 없다면 서로 다른 보안 도구 결과를 어떻게 통합할 수 있을까?
    
2. GitHub Security Tab에 취약점을 표시하면 어떤 장점이 있을까?
    
3. 모든 취약점을 SARIF로 업로드해야 할까, 아니면 Critical/High만 해야 할까?
    

---

## ✅ 정리

- SARIF는 정적 분석 및 보안 결과 공유를 위한 국제 표준 포맷
    
- Trivy 결과를 SARIF로 변환하면 GitHub Security Tab과 연계 가능
    
- 개발자·보안팀·경영진 모두 같은 인터페이스에서 보안 결과를 공유할 수 있음
---

# 10주차: Trivy 결과 → SARIF 변환 및 GitHub Security Tab 업로드 (실습, 3시간)

---

## 🎯 학습 목표

- SARIF 포맷으로 보안 결과 변환
    
- GitHub Actions에서 Trivy SARIF 리포트 생성
    
- GitHub Security Tab(Code scanning alerts)에 결과 반영
    

---

## 1️⃣ SARIF 변환 실습 (40분)

### 컨테이너 이미지 스캔 → SARIF

```bash
trivy image --format sarif -o reports/python.sarif python:3.9
```

### IaC 코드 스캔 → SARIF

```bash
trivy config --format sarif -o reports/terraform.sarif ./terraform
```

### AWS 계정 스캔 → SARIF

```bash
trivy aws --region ap-northeast-2 --format sarif -o reports/aws.sarif
```

👉 `reports/` 폴더에 `.sarif` 파일 생성

---

## 2️⃣ GitHub Actions 연동 (40분)

`.github/workflows/trivy-sarif.yml`

```yaml
name: Trivy Security Scan SARIF
on: [push]

jobs:
  trivy:
    runs-on: ubuntu-latest
    permissions:
      security-events: write  # SARIF 업로드 권한
    steps:
      - uses: actions/checkout@v3

      - name: Run Trivy image scan (SARIF)
        run: trivy image --format sarif -o trivy-image.sarif python:3.9

      - name: Upload SARIF to GitHub Security Tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: trivy-image.sarif
```

👉 실행 후 GitHub → **Security → Code scanning alerts** 확인

---

## 3️⃣ 결과 확인 및 분석 (30분)

1. GitHub Security Tab에서 취약점 목록 확인
    
2. 각 취약점 클릭 → 상세 정보 확인 (CVE, 영향도, 위치)
    
3. 실제 코드/환경에서 수정 가능한 항목 식별
    

---

## 4️⃣ 확장 미션 (40분)

- IaC (`terraform.sarif`)와 AWS 계정(`aws.sarif`)도 업로드하도록 워크플로우 확장
    
- Critical/High만 필터링해 SARIF 생성 (`--severity CRITICAL,HIGH`)
    
- 팀별로 Repo에 **SARIF 기반 Code scanning alerts 스크린샷** 업로드
    

---

## 5️⃣ 팀별 과제 (20분)

1. 팀 Repo의 **Security Tab** 화면 캡처
    
2. 발견된 Critical/High 취약점 2개 요약
    
3. 개선 방안 제시 → `reports/team_sarif_report.md` 제출
    

---

## ✅ 정리

- Trivy 결과를 SARIF 포맷으로 변환 가능
    
- GitHub Actions에서 SARIF 업로드 시 Security Tab에 자동 반영
    
- 개발자와 보안팀이 Repo 내에서 직접 보안 결과를 공유하고 개선 가능
