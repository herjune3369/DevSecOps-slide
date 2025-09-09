# 📝 11주차 실습 가이드 – GitHub Actions + Terraform + Ansible → CI/CD 자동화

---

## 🎯 학습 목표

- GitHub Actions에서 Terraform과 Ansible을 실행하는 방법을 배운다.
    
- push 이벤트가 발생하면 → **Terraform으로 인프라 배포** + **Ansible로 앱 배포**가 자동 실행되는 파이프라인을 만든다.
    
- CI/CD의 기본 개념(“코드 변경 → 자동 실행 → 결과 반영”)을 실제 인프라 환경에서 경험한다.
    

---

## 1️⃣ 전체 흐름 개요

```
[학생 코드 푸시] → [GitHub Actions Workflow 실행]
       │
       ├─ Step 1. Terraform → AWS 인프라 생성 (VPC, EC2, RDS, ALB)
       │
       └─ Step 2. Ansible → EC2에 Flask 앱 자동 배포
```

👉 이 과정을 통해 **코드만 올려도 인프라+앱 배포 완료**가 된다.

---

## 2️⃣ GitHub Secrets 설정

1. GitHub Repo → Settings → Secrets and variables → Actions → New repository secret
    
2. 다음 항목 등록:
    
    - `AWS_ACCESS_KEY_ID`
        
    - `AWS_SECRET_ACCESS_KEY`
        
    - `AWS_REGION` (예: `ap-northeast-2`)
        

👉 학생들에게 강조: **AWS 키는 절대 코드에 직접 쓰면 안 된다 → Secrets에 저장해야 한다.**

---

## 3️⃣ Workflow 파일 작성

📂 `.github/workflows/deploy.yml`

```yaml
name: CI/CD with Terraform and Ansible

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # 1. 코드 체크아웃
      - name: Checkout repository
        uses: actions/checkout@v3
      # 2. AWS 인증
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
```
---
```yaml

      # 3. Terraform 실행
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init
        working-directory: ./terraform

      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./terraform

      # 4. Ansible 실행
      - name: Install Ansible
        run: sudo apt-get update && sudo apt-get install -y ansible

      - name: Run Ansible Playbook
        run: ansible-playbook -i ansible/inventory.ini ansible/playbook.yml
```

---

## 4️⃣ 디렉토리 구조 예시

```
repo-root/
 ├─ terraform/              # Terraform 코드 (VPC, EC2, RDS, ALB)
 ├─ ansible/                # Ansible 코드 (roles, playbook)
 │   ├─ inventory.ini
 │   ├─ playbook.yml
 │   └─ roles/
 └─ .github/
     └─ workflows/
         └─ deploy.yml
```

---

## 5️⃣ 실행 시나리오

1. 로컬에서 `app.py`를 수정 후 GitHub에 push
    
2. GitHub Actions → `deploy.yml` 실행 시작
    
3. 로그에서 Terraform → AWS 인프라 적용 확인
    
4. 이어서 Ansible → Flask 앱 자동 배포
    
5. 최종적으로 ALB DNS 접속 → 변경된 웹페이지 확인
    

---

## 6️⃣ 학습 정리

- **CI/CD 핵심 개념**
    
    - CI(지속적 통합): 코드가 push되면 자동 빌드/테스트
        
    - CD(지속적 배포): 코드가 push되면 자동 배포까지
        
- **Terraform + Ansible + GitHub Actions**를 연결해보면서,  
    학생들은 “인프라 → 앱 → 배포”의 전체 파이프라인을 경험
    

👉 이번 주차를 마치면 **포트폴리오에 넣을 만한 CI/CD 자동화 프로젝트**를 확보

---

## ✅ 과제

1. ALB DNS 접속 시 Flask 앱 페이지가 정상적으로 바뀌는지 확인
    
2. GitHub Actions 실행 로그를 캡처해서 보고서에 첨부
    
3. **도전 과제**: `terraform plan`과 `terraform apply`를 Job 2개로 분리해보고, `plan` 결과를 GitHub PR 코멘트로 남기기
    


