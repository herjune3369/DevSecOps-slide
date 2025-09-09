
# 7주차: 이론 – CI/CD 파이프라인과 보안 자동화

---

## 🎯 학습 목표

- CI/CD 파이프라인의 구조와 특징을 설명할 수 있다
    
- DevSecOps에서 CI/CD와 보안 자동화가 결합되는 과정을 이해한다
    
- 보안 자동화 도구(Trivy 등)를 CI/CD에 통합해야 하는 이유를 설명할 수 있다
    

---

## 1️⃣ CI/CD 개요

### CI (Continuous Integration, 지속적 통합)

- 개발자가 작성한 코드를 **지속적으로 통합·빌드·테스트**
    
- 문제를 조기에 발견하고 빠르게 수정
    

### CD (Continuous Delivery/Deployment, 지속적 제공/배포)

- 빌드된 코드를 **자동으로 배포 환경까지 전달**
    
- 배포 속도와 안정성 강화
    

---

## 2️⃣ CI/CD 파이프라인 단계

1. **코드 커밋**: 개발자가 GitHub/GitLab에 코드 push
    
2. **빌드**: 코드 컴파일, 이미지 빌드
    
3. **테스트**: 단위 테스트, 통합 테스트 실행
    
4. **배포(Deploy)**: 스테이징 → 운영 환경 자동 반영
    
5. **모니터링**: 로그, 성능, 보안 점검
    

👉 DevOps의 핵심: 코드 변경 → 배포까지 **자동화**

---

## 3️⃣ 보안 자동화의 필요성

- 전통적 보안 점검: **배포 후** 취약점 발견 → 비용 ↑
    
- DevSecOps: **개발·빌드 단계에서 보안 점검(Shift Left Security)**
    
- 보안 자동화 없을 때 문제
    
    - 취약한 이미지가 운영 환경에 배포
        
    - IaC 코드에 잘못된 보안 그룹이 그대로 적용
        
    - 계정 보안 설정이 감사 직전까지 방치
        

---

## 4️⃣ CI/CD + 보안 자동화 흐름

1. **CI 단계**
    
    - `trivy image` → 컨테이너 이미지 점검
        
    - `trivy config` → IaC 코드 점검
        
2. **CD 단계**
    
    - 배포 전 `trivy aws` → 계정 정책 확인
        
    - `.trivyignore` → False Positive 관리
        
3. **결과 처리**
    
    - JSON → Markdown/PDF 리포트
        
    - SARIF 변환 → GitHub Security Tab 반영
        

---

## 5️⃣ 도구와 사례

- **Trivy**: 컨테이너 이미지, IaC, 계정 점검
    
- **Checkov**: IaC 코드 규정 준수 점검
    
- **SonarQube**: 코드 품질 + 보안 점검
    
- **GitHub Actions**: 오픈소스 CI/CD 플랫폼
    

사례:

- 스타트업 → GitHub Actions + Trivy, PR 생성 시 자동 스캔
    
- 금융사 → Jenkins + SonarQube/Trivy, 빌드 단계에서 보안 점검
    

---

## 6️⃣ 토론 과제

1. CI/CD 파이프라인에서 **보안 자동화**를 빼면 어떤 문제가 발생할까?
    
2. CSPM, CWPP 점검을 CI 단계와 CD 단계 중 어디에 넣는 게 효과적일까?
    
3. 보안 자동화 결과 리포트는 개발자와 보안팀 중 누구에게 어떻게 전달해야 할까?
    

---

## ✅ 정리

- CI/CD는 코드 커밋부터 배포까지 자동화하는 파이프라인
    
- 보안 자동화는 DevSecOps의 핵심 → Shift Left Security 구현
    
- Trivy 같은 도구를 CI/CD에 통합하면, 코드·이미지·계정 단계 모두에서 자동 점검 가능
    
- 결과를 자동 리포트화하여 개발팀과 보안팀 모두 활용할 수 있음
    

👉 다음 주(8주차 이론): **False Positive와 보안 리포트 신뢰성 관리**

---
# 7주차: GitHub Actions에 Trivy 보안 스캔 연동 (실습, 3시간)

---

## 🎯 학습 목표

- GitHub Actions 워크플로우 구조 이해
    
- Trivy를 CI/CD 파이프라인에 연동해 자동 스캔 수행
    
- 보안 스캔 결과를 저장·활용하는 방법 학습
    

---

## 1️⃣ GitHub Actions 기초 확인 (20분)

- 워크플로우: `.github/workflows/*.yml`
    
- 트리거: `on: push`, `on: pull_request`
    
- 잡(Job): 실행 환경 (`runs-on: ubuntu-latest`)
    
- 스텝(Step): 코드 체크아웃, Trivy 실행, 결과 저장
    

👉 실습 전 Repo 구조 확인

```bash
cd devsecops-sample
tree -L 2
```

```
.github/workflows/
terraform/
ansible/
reports/
```

---
<!-- _class: medium -->

## 2️⃣ Trivy 워크플로우 작성 (40분)

`.github/workflows/trivy.yml`

```yaml
name: Trivy Security Scan
on: [push]

jobs:
  trivy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Run Trivy image scan
        run: trivy image python:3.9

      - name: Save JSON Report
        run: trivy image -f json -o trivy.json python:3.9

      - name: Upload Report Artifact
        uses: actions/upload-artifact@v3
        with:
          name: trivy-report
          path: trivy.json
```

👉 첫 번째 자동 보안 스캔 파이프라인 완성

---

## 3️⃣ 실행 및 결과 확인 (30분)

1. GitHub → **Actions 탭** → 실행 내역 클릭
    
2. `Run Trivy image scan` 로그에서 취약점 확인
    
3. **Artifacts** → `trivy-report` 다운로드
    
4. JSON 결과 열기
    

```bash
cat trivy.json | jq '.Results[0].Vulnerabilities[0]'
```

예상 로그

```
python:3.9 (debian 11.7)
Total: 12 (LOW: 3, MEDIUM: 5, HIGH: 3, CRITICAL: 1)
```

---

## 4️⃣ 확장 실습 – IaC & AWS 점검 추가 (40분)

워크플로우에 Step 추가

```yaml
- name: Run Trivy IaC scan
  run: trivy config -f json -o trivy-config.json ./terraform

- name: Run Trivy AWS scan
  run: trivy aws --region ap-northeast-2 -f json -o trivy-aws.json

- name: Upload All Reports
  uses: actions/upload-artifact@v3
  with:
    name: trivy-reports
    path: |
      trivy.json
      trivy-config.json
      trivy-aws.json
```

👉 이제 컨테이너, IaC, AWS 계정을 동시에 점검

---

## 5️⃣ 팀별 과제 (30분)

1. 각 팀은 `.yml` 수정 → 세 가지 스캔 통합
    
2. 실행 후 JSON 3종 다운로드
    
3. Critical 취약점 1개 분석 → Markdown 보고서 작성
    
    - 취약점 설명
        
    - 영향도
        
    - 개선 방법
        

---

## 6️⃣ 발표 & 토론 (20분)

- 팀별 발표: 보고서 5분 공유
    
- 비교: 공통 취약점 vs 특이 취약점
    
- 토론 주제:
    
    - CI 단계 vs CD 단계, 어느 시점에 스캔을 넣는 게 효과적인가?
        
    - False Positive는 누구(개발자/보안팀)가 관리해야 하는가?
        

---

## ✅ 정리

- GitHub Actions에 Trivy 스캔을 연동했다
    
- 컨테이너, IaC, AWS 계정 점검을 자동화했다
    
- JSON 리포트를 Artifacts로 저장하고 공유하는 방법을 배웠다
    

👉 다음 주(8주차 실습): **`.trivyignore` 작성 및 False Positive 관리**

