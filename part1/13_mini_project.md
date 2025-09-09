
# 📝 13주차 실습 가이드 – GitHub Actions + Terraform + Ansible → CI/CD 자동화

---

## 🎯 학습 목표

- GitHub Actions에서 Terraform과 Ansible을 실행하는 방법을 배운다.
    
- push 이벤트가 발생하면 → **Terraform으로 인프라 배포** + **Ansible로 앱 배포**가 자동 실행되는 파이프라인을 만든다.
    
- GitHub에 코드를 올리고 Actions 실행까지 연결되는 전체 DevOps 자동화 흐름을 경험한다.
    

---

## 1️⃣ 전체 흐름 개요

```
[로컬 PC] 코드 수정 + git push
        ↓
[GitHub Repo] 새 커밋 → Actions Workflow 실행
        ↓
[Job 1] Terraform → AWS 인프라 배포 (VPC, EC2, RDS, ALB)
        ↓
[Job 2] Ansible → EC2에 Flask 앱 자동 배포
        ↓
[사용자] ALB DNS 접속 → 변경된 앱 확인
```

---

## 2️⃣ GitHub Secrets 설정

1. GitHub Repo → Settings → Secrets and variables → Actions → New repository secret
    
2. 다음 값 등록:
    
    - `AWS_ACCESS_KEY_ID`
        
    - `AWS_SECRET_ACCESS_KEY`
        
    - `AWS_REGION` (예: `ap-northeast-2`)
        

👉 학생들에게 강조: **AWS 키는 절대 코드에 직접 쓰지 않는다 → 반드시 Secrets에 저장!**

---

## 3️⃣ Terraform Output 정의

📂 **terraform/outputs.tf**

```hcl
output "ec2_public_ip" {
  value = aws_instance.web.public_ip
}

output "rds_endpoint" {
  value = aws_db_instance.mysql.endpoint
}

output "alb_dns_name" {
  value = aws_lb.app_alb.dns_name
}
```

---

## 4️⃣ Workflow 작성

📂 **.github/workflows/deploy.yml**

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
```
---
```yaml
      # 2. AWS 인증
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      # 3. Terraform 실행
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init
        working-directory: ./terraform

      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./terraform
```
---
```yaml
      # 4. Terraform Output 추출
      - name: Get Terraform Outputs
        id: tf-out
        run: |
          cd terraform
          echo "ec2_public_ip=$(terraform output -raw ec2_public_ip)" >> $GITHUB_ENV
          echo "rds_endpoint=$(terraform output -raw rds_endpoint)" >> $GITHUB_ENV
          echo "alb_dns_name=$(terraform output -raw alb_dns_name)" >> $GITHUB_ENV

      # 5. Ansible 실행
      - name: Install Ansible
        run: sudo apt-get update && sudo apt-get install -y ansible

      - name: Run Ansible Playbook
        run: |
          ansible-playbook -i ansible/inventory.ini ansible/playbook.yml \
            --extra-vars "rds_endpoint=${{ env.rds_endpoint }} ec2_public_ip=${{ env.ec2_public_ip }}"
```

---

<!-- _class: medium -->

## 5️⃣ GitHub에 코드 올리기

### (1) Git 초기화 & 첫 커밋

```bash
cd ~/devsecops-lab/politec-book   # 예시 프로젝트 디렉토리
git init
git add .
git commit -m "First commit - Terraform + Ansible + CI/CD pipeline"
```

### (2) 원격 Repo 연결

```bash
git branch -M main
git remote add origin https://github.com/<username>/devops-cicd-demo.git
```

### (3) GitHub에 푸시

```bash
git push -u origin main
```

👉 GitHub Repo에 코드가 올라가고,  `.github/workflows/deploy.yml`이 있으므로 **Actions 자동 실행**!

---

## 6️⃣ 실행 흐름 그림

```
로컬 PC                GitHub                 AWS
--------              --------               -----
app.py 수정  → push →  Repo(main) → Actions → Terraform → EC2, RDS, ALB
                                     ↓
                                  Ansible → EC2에 Flask 앱 배포
                                     ↓
                               ALB DNS → 웹 브라우저에서 확인
```

---

## 7️⃣ 학습 정리

- GitHub Actions는 **push 이벤트**를 자동 감지한다.
    
- Terraform으로 인프라 생성 → output 추출 → Ansible에 전달 → 앱 배포까지 자동으로 연결된다.
    
- 학생들은 이제 **로컬에서 코드만 수정하고 push하면 전체 배포가 된다**는 DevOps 자동화의 핵심을 경험한다.
    

---

## ✅ 과제

1. `terraform/outputs.tf`에 `alb_dns_name` 출력 추가
    
2. GitHub Actions 로그에서 `alb_dns_name`을 확인하고, 실제 DNS로 접속해 앱 페이지 확인
    
3. 실행 로그와 브라우저 접속 화면을 스크린샷해서 보고서에 첨부
    



