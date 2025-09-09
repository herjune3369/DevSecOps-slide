
# 🧑‍🏫 14주차 이론 강의 – 보안 리포트 자동 생성 스크립트 활용

## 1. DevSecOps에서 보안 리포트의 필요성

- 보안 도구(Trivy, Gatekeeper, Falco 등)는 많은 진단 결과를 생성
    
- 그대로는 **데이터 과부하** → 관리자가 빠르게 이해하기 어려움
    
- **자동 보고서 생성 스크립트**로 결과를 요약·시각화해야 함
    
- 목적:
    
    - **개발자** → 수정해야 할 취약점
        
    - **보안팀** → 정책 위반 내역
        
    - **경영진** → 종합 요약(High Risk 개수, 대응 현황)
        

---

## 2. `generate_security_report.py` 개요

- Python 기반 보안 리포트 자동화 스크립트
    
- 입력: Trivy, Falco, Gatekeeper 등에서 생성한 JSON 결과 파일
    
- 처리: 파싱 → 취약점 심각도별 분류 → Markdown/PDF 요약
    
- 출력:
    
    - Markdown (`report.md`)
        
    - PDF (`report.pdf`)
        
    - HTML 대시보드(선택)
        

---
<!-- _class: medium -->

## 3. 스크립트 동작 구조

### 입력 (JSON 예시 – Trivy)

```json
{
  "Target": "nginx:1.25",
  "Vulnerabilities": [
    { "VulnerabilityID": "CVE-2023-12345", "PkgName": "openssl", "Severity": "HIGH" },
    { "VulnerabilityID": "CVE-2023-67890", "PkgName": "zlib", "Severity": "MEDIUM" }
  ]
}
```

### 출력 (Markdown 예시)

```markdown
# Security Report

## Summary
- HIGH: 1
- MEDIUM: 1
- LOW: 0

## Details
- [HIGH] CVE-2023-12345 (openssl)
- [MEDIUM] CVE-2023-67890 (zlib)
```

---

## 4. 핵심 기능

- **취약점 요약**
    
    - HIGH/MEDIUM/LOW 개수 집계
        
- **세부 내역 정리**
    
    - 패키지명, CVE ID, 심각도
        
- **포맷 변환**
    
    - Markdown → PDF (wkhtmltopdf 등 사용)
        
- **자동화 통합**
    
    - GitHub Actions에서 Trivy 실행 후 → Python 스크립트 실행 → Report 업로드
        

---

## 5. CI/CD 파이프라인 통합

1. **Trivy 실행** → JSON 출력 저장
    
2. **Python 스크립트 실행** → Markdown/PDF 보고서 생성
    
3. **Artifacts 업로드** → GitHub Actions 빌드 결과로 첨부
    
4. 선택: Slack, Teams, 이메일로 자동 전송
    

---

## 6. 실무 활용 예시

- 매일 새벽 Trivy + Falco 결과 자동 리포트 생성
    
- GitHub Pull Request 시 보안 리포트 코멘트 자동 추가
    
- 관리자는 **대시보드/보고서만 확인**하고, 세부 수정은 개발팀에 전달
    

---

## ✅ 정리

- DevSecOps 환경에서는 보안 점검 결과를 **자동 보고서화** 해야 한다.
    
- `generate_security_report.py`는 JSON → Markdown/PDF로 변환하는 핵심 도구
    
- CI/CD와 연계하면 **자동 점검 + 자동 보고**가 가능
    
- 다음(14주차 실습)에서는 실제로 스크립트를 실행하여 보안 리포트를 생성해본다.
    

---

# 💻 14주차 실습 강의 – 보안 리포트 자동 생성 스크립트 활용 (`generate_security_report.py`)

## 🎯 실습 목표

- Trivy를 사용해 컨테이너 이미지를 스캔하고 JSON 결과를 저장한다.
    
- Python 스크립트를 실행하여 Markdown/PDF 형식의 보안 보고서를 자동 생성한다.
    
- CI/CD 파이프라인(GitHub Actions)과 연동하여 자동 보고 체계를 체험한다.
    

---

## 1. Trivy로 이미지 스캔 결과 생성

### 1-1. nginx 이미지 취약점 스캔

```bash
trivy image -f json -o trivy-report.json nginx:1.25
```

### 1-2. 결과 파일 확인

```bash
cat trivy-report.json | jq '.Results[0].Vulnerabilities[0]'
```

✅ JSON 출력 예시:

```json
{
  "VulnerabilityID": "CVE-2023-12345",
  "PkgName": "openssl",
  "InstalledVersion": "1.1.1k",
  "FixedVersion": "1.1.1u",
  "Severity": "HIGH"
}
```

---

<!-- _class: small -->

## 2. Python 스크립트 작성

### 2-1. `generate_security_report.py`

```python
import json
from collections import Counter
from fpdf import FPDF

# 1. JSON 로드
with open("trivy-report.json") as f:
    data = json.load(f)

vulns = []
for result in data.get("Results", []):
    for v in result.get("Vulnerabilities", []):
        vulns.append({
            "id": v["VulnerabilityID"],
            "pkg": v["PkgName"],
            "severity": v["Severity"]
        })

# 2. 요약 통계
severity_count = Counter(v["severity"] for v in vulns)

# 3. Markdown 보고서 작성
with open("security_report.md", "w") as f:
    f.write("# Security Report\n\n")
    f.write("## Summary\n")
    for sev, count in severity_count.items():
        f.write(f"- {sev}: {count}\n")
    f.write("\n## Details\n")
    for v in vulns:
        f.write(f"- [{v['severity']}] {v['id']} ({v['pkg']})\n")

# 4. PDF 보고서 작성
pdf = FPDF()
pdf.add_page()
pdf.set_font("Arial", size=12)
pdf.cell(200, 10, txt="Security Report", ln=True, align="C")

pdf.ln(10)
pdf.set_font("Arial", size=10)
for sev, count in severity_count.items():
    pdf.cell(200, 10, txt=f"{sev}: {count}", ln=True)

pdf.ln(10)
for v in vulns:
    pdf.cell(200, 10, txt=f"[{v['severity']}] {v['id']} ({v['pkg']})", ln=True)

pdf.output("security_report.pdf")
print("Reports generated: security_report.md, security_report.pdf")
```

---
### 2-2. 의존성 설치

```bash
pip install fpdf
```

---

## 3. 스크립트 실행

```bash
python3 generate_security_report.py
```

✅ 출력:

```
Reports generated: security_report.md, security_report.pdf
```

---

## 4. 결과 확인

- `security_report.md` → 요약 및 상세 취약점 목록 (텍스트)
    
- `security_report.pdf` → 이식성 높은 보안 보고서
    

예시 (Markdown):

```
# Security Report

## Summary
- HIGH: 3
- MEDIUM: 5
- LOW: 2

## Details
- [HIGH] CVE-2023-12345 (openssl)
- [MEDIUM] CVE-2023-67890 (zlib)
```

---

<!-- _class: small -->

## 5. GitHub Actions 연동

### 5-1. `.github/workflows/security-report.yaml`

```yaml
name: Security Report

on:
  push:
    branches: [ "main" ]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install Trivy
        run: |
          sudo apt-get install wget -y
          wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.0_Linux-64bit.deb
          sudo dpkg -i trivy_0.50.0_Linux-64bit.deb

      - name: Run Trivy
        run: trivy image -f json -o trivy-report.json nginx:1.25

      - name: Install Python dependencies
        run: pip install fpdf

      - name: Generate Report
        run: python3 generate_security_report.py

      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: security-reports
          path: |
            security_report.md
            security_report.pdf
```

---

## 6. 정리

리소스 정리 불필요 (보고서만 생성)

---

## ✅ 최종 결과

- Trivy로 이미지 보안 스캔 → JSON 결과 확보
    
- Python 스크립트 실행 → Markdown & PDF 보고서 자동 생성
    
- GitHub Actions와 연계 → **보안 점검 + 자동 보고** 파이프라인 완성
    
