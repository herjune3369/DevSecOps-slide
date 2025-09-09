
# 11주차: 이론 – Mini Project ②, DevSecOps 파이프라인 완성

---

## 🎯 학습 목표

- DevSecOps 파이프라인의 전체 구성 요소를 이해한다
    
- Terraform, Ansible, GitHub Actions, Trivy가 어떻게 통합되는지 설명할 수 있다
    
- 보안 자동화 파이프라인 완성의 의미와 기대 효과를 이해한다
    

---
<!-- _class: medium -->

## 1️⃣ DevSecOps 파이프라인 구성요소

1. **Terraform (IaC)**
    
    - 인프라 자동화 → VPC, EC2, RDS, ALB 등 코드로 정의
        
    - 보안 그룹, 네트워크 설정까지 코드화
        
2. **Ansible (Configuration Management)**
    
    - 애플리케이션 배포 및 환경 설정 자동화
        
    - Flask 앱 설치, DB 초기화, 패키지 업데이트
        
3. **GitHub Actions (CI/CD)**
    
    - 코드 변경 → 자동 빌드, 배포
        
    - 워크플로우 내에 보안 스캔 삽입 가능
        
4. **Trivy (Security Scanner)**
    
    - CWPP: 컨테이너 이미지 취약점 점검
        
    - CSPM: IaC 코드 및 AWS 계정 설정 점검
        
    - SARIF/Markdown/PDF 리포트 생성
        

---

<!-- _class: medium -->

## 2️⃣ 통합 파이프라인 동작 흐름

1. **개발자 Push → GitHub Repo**
    
    - 코드 변경 감지
        
2. **GitHub Actions Trigger**
    
    - Terraform 적용 (인프라 생성/수정)
        
    - Ansible 실행 (앱 자동 배포)
        
3. **보안 스캔 자동화**
    
    - `trivy image` → 컨테이너 취약점
        
    - `trivy config` → IaC 코드 보안 점검
        
    - `trivy aws` → 계정 설정 점검
        
4. **결과 보고**
    
    - JSON → Markdown/PDF 변환
        
    - SARIF 업로드 → GitHub Security Tab 반영
        

---

## 3️⃣ DevSecOps 파이프라인 완성의 의미

- **Shift Left Security**
    
    - 개발 단계에서부터 보안 점검 자동화
        
- **속도와 보안의 균형**
    
    - 빠른 배포 + 안전한 배포 동시 달성
        
- **협업 효율화**
    
    - 개발자, 보안팀, 운영팀이 동일한 리포트를 공유
        
- **규제 및 감사 대응**
    
    - 자동 리포트를 증적 자료로 제출 가능
        

---

## 4️⃣ 기대 효과

- **개발자**: 배포 전 보안 피드백 즉시 확인 → 수정 용이
    
- **보안팀**: Critical/High 취약점만 선별 대응 가능
    
- **경영진**: DevSecOps 파이프라인 자체가 “보안 내재화”를 증명
    
- **조직 전반**:
    
    - 장애 발생률 감소
        
    - 보안 사고 예방
        
    - DevOps 문화 성숙
        

---

## 5️⃣ 토론 과제

1. DevOps와 DevSecOps 파이프라인의 가장 큰 차이는 무엇인가?
    
2. 우리 팀에 적용한다면, 파이프라인에서 어떤 단계(코드, IaC, 컨테이너, 계정)를 **우선순위**로 강화해야 할까?
    
3. 보안팀과 개발팀이 **하나의 자동화 파이프라인**을 공유하면 어떤 장단점이 있을까?
    

---

## ✅ 정리

- DevSecOps 파이프라인은 Terraform + Ansible + GitHub Actions + Trivy로 완성된다
    
- 코드/인프라/워크로드/계정 모든 단계를 자동 점검
    
- 자동화된 보안 리포트는 개발·보안·경영진 모두에게 유용하다
---

# 11주차: Mini Project ② – Trivy 중심 DevSecOps 파이프라인 완성 (실습, 3시간)

---

## 🎯 학습 목표

- Terraform + Ansible + GitHub Actions + Trivy를 하나의 파이프라인으로 통합한다
    
- 자동 인프라 배포 → 앱 배포 → 보안 스캔 → 리포트 자동화를 구현한다
    
- 팀별로 완성된 파이프라인을 테스트하고 결과를 공유한다
    

---

## 1️⃣ 프로젝트 개요 (20분)

- **목표**: DevSecOps 파이프라인 완성
    
    1. Terraform → AWS 인프라 배포
        
    2. Ansible → 앱 자동 배포
        
    3. Trivy → 이미지/IaC/AWS 보안 점검
        
    4. GitHub Actions → 전체 자동화 + 리포트 생성
        

👉 업로드된 코드 구조(`terraform/`, `ansible/`, `.github/workflows/`, `scan-report/`)를 기반으로 진행

---

## 2️⃣ Terraform으로 인프라 배포 (40분)

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

- 배포 리소스: VPC, Subnet, EC2, RDS, Security Group
    
- 출력 확인:
    

```bash
terraform output
```

👉 `ec2_public_ip` 확인 후 Ansible에 반영

---

## 3️⃣ Ansible로 앱 배포 (40분)

인벤토리(`ansible/inventories/hosts.ini`) 수정

```
[app]
<ec2_public_ip>
```

플레이북 실행

```bash
cd ansible
ansible-playbook -i inventories/hosts.ini playbook.yml
```

👉 Flask 앱 배포 확인: `http://<ec2_public_ip>:5000`

---

## 4️⃣ Trivy 보안 스캔 통합 (30분)

### 컨테이너 이미지 점검

```bash
trivy image -f json -o reports/python.json python:3.9
```

### IaC 점검

```bash
trivy config -f json -o reports/terraform.json ./terraform
```

### AWS 계정 점검

```bash
trivy aws --region ap-northeast-2 -f json -o reports/aws.json
```

---
<!-- _class: small -->

## 5️⃣ GitHub Actions 자동화 (30분)

`.github/workflows/devsecops.yml`

```yaml
name: DevSecOps Pipeline
on: [push]

jobs:
  devsecops:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v3

      - name: Terraform Apply
        run: |
          cd terraform
          terraform init
          terraform apply -auto-approve

      - name: Ansible Deploy
        run: |
          cd ansible
          ansible-playbook -i inventories/hosts.ini playbook.yml

      - name: Trivy Image Scan
        run: trivy image -f json -o reports/python.json python:3.9

      - name: Trivy IaC Scan
        run: trivy config -f json -o reports/terraform.json ./terraform

      - name: Trivy AWS Scan
        run: trivy aws --region ap-northeast-2 -f json -o reports/aws.json

      - name: Upload Reports
        uses: actions/upload-artifact@v3
        with:
          name: security-reports
          path: reports/
```

---

## 6️⃣ 팀별 과제 (20분)

1. 각 팀 Repo에서 CI 실행 → 전체 파이프라인 자동화
    
2. `reports/` 내 JSON 결과 확인
    
3. Top 3 취약점 정리 + 개선 방안 Markdown 보고서 작성
    
4. 팀 Repo에 업로드 후 발표 준비
    

---

## 7️⃣ 발표 & 토론 (20분)

- 팀별 발표: “파이프라인 실행 결과 + 주요 취약점 + 개선 방안”
    
- 토론 주제:
    
    - 자동화된 DevSecOps 파이프라인의 장점은?
        
    - 우리 조직이라면 어디까지 자동화할 수 있을까?
        

---

## ✅ 정리

- Terraform + Ansible + GitHub Actions + Trivy를 통합 → DevSecOps 파이프라인 완성
    
- 자동 배포 + 자동 보안 점검 + 자동 리포트 생성 가능
    
- 실무에서도 그대로 적용 가능한 **End-to-End DevSecOps 흐름** 체험
    

---
