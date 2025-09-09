
# 14주차: 최종 보고서 및 산출물 문서화 (실습, 3시간)

---

## 🎯 학습 목표

- Trivy JSON 결과를 Markdown/PDF/SARIF로 변환한다
    
- GitHub Actions에 SARIF 업로드를 연동한다
    
- README/아키텍처/보안 리포트를 포함해 최종 Repo를 정리한다
    

---
<!-- _class: medium -->

## 1️⃣ 보안 리포트 자동 생성 (40분)

### Python 스크립트 실행

```bash
python generate_security_report.py \
  --inputs reports/python.json reports/terraform.json reports/aws.json \
  --output reports/final_report.md
```

👉 결과물:

- `reports/final_report.md`
    
- `reports/final_report.pdf`
    
---
### Markdown 결과 예시

```markdown
# Security Scan Report

## Container (CWPP)
- python:3.9 → Critical 1, High 3

## IaC (CSPM)
- SG 0.0.0.0/0 허용 (HIGH)

## AWS Account (CSPM)
- Root MFA 미사용 (HIGH)
- S3 Public Access 허용 (CRITICAL)

## 종합 분석
- Top 3 위험:
  1. S3 Public Access (CRITICAL)
  2. Root MFA 미사용 (HIGH)
  3. OpenSSL 취약점 (CRITICAL)
```

---
<!-- _class: medium -->

## 2️⃣ SARIF 변환 및 Security Tab 업로드 (40분)

### 로컬 SARIF 변환

```bash
trivy image --format sarif -o reports/python.sarif python:3.9
trivy config --format sarif -o reports/terraform.sarif ./terraform
trivy aws --region ap-northeast-2 --format sarif -o reports/aws.sarif
```

### GitHub Actions 워크플로우 (`.github/workflows/security.yml`)

```yaml
name: Trivy SARIF Upload
on: [push]

jobs:
  trivy:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v3

      - name: Run Trivy Image Scan (SARIF)
        run: trivy image --format sarif -o trivy-image.sarif python:3.9

      - name: Upload SARIF to GitHub Security Tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: trivy-image.sarif
```

👉 실행 후: GitHub → **Security → Code scanning alerts** 에 결과 표시

---

<!-- _class: medium -->

## 3️⃣ Repo 문서화 (40분)

### README.md 기본 구조

```markdown
# DevSecOps Final Project – AI 사주 앱

## 🎯 목표
- Terraform + Ansible + GitHub Actions + Trivy로 DevSecOps 파이프라인 완성

## 🏗️ 아키텍처
(다이어그램 이미지 삽입)

## 🔑 주요 기능
- IaC 배포 (Terraform)
- 자동 배포 (Ansible)
- 보안 스캔 (Trivy – CWPP, CSPM)
- 자동 리포트 (Markdown, PDF, SARIF)

## 📊 보안 리포트 요약
- python:3.9 → Critical 1
- IaC SG 오픈 → High 1
- AWS Root MFA 미사용 → High 1

## 📂 Repo 구조
```

app/  
terraform/  
ansible/  
.github/workflows/  
reports/

---
````

## 🚀 실행 방법
```bash
terraform apply
ansible-playbook -i inventories/hosts.ini playbook.yml
````

## 👥 팀원 & 역할

- 개발: ○○○
    
- 인프라: ○○○
    
- 보안: ○○○
    
- 리더: ○○○
    
---
````

---

## 4️⃣ 최종 정리 (30분)
- 모든 코드/리포트 최신화 후 main 브랜치 푸시
```bash
git add .
git commit -m "Finalize project with reports and docs"
git push origin main
````

- Repo에 `reports/final_report.pdf`, `reports/*.sarif`, `README.md` 포함 확인
    

---

## 5️⃣ 팀별 과제 (30분)

1. 팀 Repo에 최종 보고서(Markdown+PDF+SARIF) 업로드
    
2. README 완성본 제출
    
3. 아키텍처 다이어그램 추가 (draw.io, excalidraw, lucidchart 등 자유)
    

---

## ✅ 정리

- JSON → Markdown/PDF/SARIF 자동 변환 완료
    
- GitHub Security Tab 연동으로 취약점 Repo 내부에서 관리
    
- 최종 Repo 문서화 및 정리 → 포트폴리오용 제출 가능
    
