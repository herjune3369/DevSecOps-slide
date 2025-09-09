
# 12주차: 이론 – 팀별 보안 자동화 프로젝트 기획

---

## 🎯 학습 목표

- 팀 단위 보안 자동화 프로젝트 기획의 중요성을 이해한다
    
- 프로젝트 기획 시 고려해야 할 요소(목표, 범위, 역할, 산출물)를 설명할 수 있다
    
- 실제 코드 기반 프로젝트(예: AI 사주 앱, Flask 쇼핑몰) 확장 아이디어를 구상한다
    

---

## 1️⃣ 왜 팀별 프로젝트인가?

- **실습 통합**: 지금까지 배운 Terraform, Ansible, GitHub Actions, Trivy를 한데 모아 실제 사례 적용
    
- **협업 경험**: DevSecOps는 본질적으로 협업 문화 → 개발/운영/보안 역할 분담 필요
    
- **포트폴리오 강화**: 개인 과제보다 팀 프로젝트 결과물이 면접·취업에서 더 강력한 증거
    

---
<!-- _class: medium -->

## 2️⃣ 프로젝트 기획 요소

1. **목표 정의**
    
    - 예: “Flask 쇼핑몰 앱 배포 및 보안 자동화 파이프라인 구축”
        
    - 예: “AI 사주 앱 보안 강화 및 Trivy 기반 리포트 자동화”
        
2. **범위 설정**
    
    - CSPM (IaC 보안 점검)
        
    - CWPP (컨테이너 보안 점검)
        
    - CI/CD 보안 자동화 (GitHub Actions)
        
    - 리포트 생성 및 공유 (Markdown, PDF, SARIF)
        
3. **역할 분담**
    
    - **개발 담당**: 애플리케이션 코드 유지
        
    - **운영 담당**: Terraform/Ansible로 인프라 자동화
        
    - **보안 담당**: Trivy 스캔 및 리포트 분석
        
    - **리더**: 통합 문서화 및 최종 발표 준비
        
4. **산출물**
    
    - GitHub Repo (코드/워크플로우/리포트)
        
    - 보안 스캔 리포트 (Markdown, PDF)
        
    - 최종 발표 자료 (슬라이드/데모 영상)
        

---
<!-- _class: medium -->

## 3️⃣ 프로젝트 예시

- **AI 사주 앱 보안 자동화**
    
    - Flask + MySQL 배포
        
    - Terraform으로 인프라 자동화
        
    - Ansible로 앱 배포
        
    - Trivy로 CSPM + CWPP 점검
        
    - GitHub Actions로 CI/CD + 보안 스캔 리포트
        
- **Flask 쇼핑몰 보안 자동화**
    
    - 사용자 로그인/상품 등록 기능 포함
        
    - IaC 코드와 Docker 이미지 보안 점검
        
    - `.trivyignore` 적용 및 False Positive 관리
        
    - SARIF 리포트를 GitHub Security Tab 업로드
        

---

## 4️⃣ 기획 단계 워크플로우

1. 팀 주제 선정
    
2. 범위 확정 (어떤 보안 기능까지 할지)
    
3. GitHub Repo 초기화 & 역할 분담
    
4. 실행 계획 수립 (12~15주차 단계별 목표)
    
5. 중간 점검: 코드 리뷰 & 리포트 공유
    

---

## 5️⃣ 토론 과제

1. 여러분의 팀은 어떤 주제로 프로젝트를 기획할 것인가?
    
2. CSPM, CWPP, CI/CD 중 가장 먼저 집중해야 할 영역은 무엇인가?
    
3. 최종 발표에서 어떤 점을 강조하면 포트폴리오 경쟁력이 높아질까?
    

---

## ✅ 정리

- 팀별 프로젝트는 **DevSecOps 전체 흐름**을 종합하는 단계
    
- 목표, 범위, 역할, 산출물을 명확히 해야 성공적인 결과 가능
    
- 실습 예제(Flask 앱, AI 사주 앱)를 확장해 자신들만의 프로젝트 기획 필요
---

# 12주차: 팀별 보안 자동화 프로젝트 기획 (실습, 3시간)

---

## 🎯 학습 목표

- 팀별로 DevSecOps 프로젝트 주제를 선정한다
    
- 목표·범위·역할·산출물을 구체화한다
    
- GitHub Repo를 기반으로 프로젝트 기획서를 작성한다
    

---

## 1️⃣ 프로젝트 주제 선정 (30분)

### 예시 1. AI 사주 앱 보안 강화

- Flask + MySQL 앱 배포
    
- Terraform + Ansible로 자동화
    
- Trivy로 IaC + 이미지 + AWS 계정 보안 점검
    
- Markdown/PDF 리포트 자동화
    

### 예시 2. Flask 쇼핑몰 보안 자동화

- 로그인·상품 기능 포함
    
- IaC 코드 및 Docker 이미지 점검
    
- `.trivyignore` 활용 False Positive 관리
    
- SARIF 리포트 GitHub Security Tab 반영
    

👉 각 팀은 **1개 주제**를 선택하거나 새롭게 제안

---

## 2️⃣ 목표와 범위 설정 (40분)

팀별로 아래 항목을 작성

1. **목표**: 프로젝트 완성 후 달성하고 싶은 보안 수준
    
2. **범위**: CSPM, CWPP, CI/CD 중 어떤 영역까지 커버할지
    
3. **성과지표**:
    
    - 취약점 80% 이상 개선
        
    - 리포트 자동화 100% 실행 성공
        
    - GitHub Security Tab 경고 확인 가능
        

👉 실습 시간: 팀별 화이트보드/Markdown 문서로 기획안 정리

---

## 3️⃣ 역할 분담 (30분)

- **개발 담당**: Flask 앱 유지·기능 추가
    
- **인프라 담당**: Terraform, Ansible 관리
    
- **보안 담당**: Trivy 스캔, `.trivyignore`, 리포트 자동화
    
- **리더**: 전체 진행 관리, 최종 발표 준비
    

👉 팀별 GitHub Repo에 `README.md`에 역할 표로 기록

---

<!-- _class: medium -->

## 4️⃣ GitHub Repo 초기화 (40분)

```bash
git clone https://github.com/<team-org>/<team-project>.git
cd <team-project>
mkdir terraform ansible reports .github/workflows
touch README.md
```

README 기본 구조

```markdown
# Team Project: DevSecOps 자동화

## 🎯 목표
(팀 목표 작성)

## 📦 범위
- CSPM: Terraform IaC 점검
- CWPP: 컨테이너 이미지 점검
- CI/CD: GitHub Actions + 리포트 자동화

## 👥 역할 분담
- 개발: OOO
- 인프라: OOO
- 보안: OOO
- 리더: OOO
```

---

## 5️⃣ 중간 점검 미션 (30분)

- 팀 Repo에 기획안과 역할 분담이 반영된 `README.md` 푸시
    
- 강사 확인 후 **피드백 제공**
    
- 다른 팀 Repo를 살펴보고 **좋은 아이디어** 피드백
    

---

## 6️⃣ 발표 & 토론 (20분)

- 각 팀 5분 발표:
    
    - 프로젝트 주제
        
    - 목표와 범위
        
    - 역할 분담
        
- 토론 주제:
    
    - “프로젝트 성공을 위해 가장 먼저 해결해야 할 보안 과제는?”
        

---

## ✅ 정리

- 팀별 주제·목표·범위·역할을 구체화하고 GitHub Repo에 반영
    
- DevSecOps 파이프라인 전체를 아우르는 프로젝트 설계 시작
    
- 이후 13~15주차는 **구현 → 테스트 → 문서화 → 발표** 단계로 진행


