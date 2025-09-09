
# 14주차 실습 가이드 – Mini Project ③: AI 사주 앱 CI/CD 완성

---

## 🎯 학습 목표

- Terraform + Ansible + GitHub Actions을 통합한 **CI/CD 파이프라인 완성**
    
- 실제로 **코드 변경 → GitHub push → AWS 배포** 전 과정을 경험
    
- 개인 또는 팀 단위로 프로젝트를 시연 및 발표
    
- 최종적으로 **GitHub Repo + 발표자료 + 보고서**를 포트폴리오로 제출
    

---

## 1️⃣ 이번 주 진행 흐름

1. **프로젝트 코드 점검**
    
    - Flask 앱 + Terraform 코드 + Ansible Role + Workflow 확인
        
    - `git status`, `terraform plan`, `ansible-playbook --check` 등으로 마지막 점검
        
2. **코드 수정 및 GitHub Push**
    
    - `app.py` 내용 일부 변경 (예: 페이지 문구 수정)
        
    - GitHub push → Actions 자동 실행 → 배포 확인
---


        
3. **ALB DNS 접속 테스트**
    
    - 실제 웹페이지가 수정된 내용으로 표시되는지 확인
        
4. **최종 발표 & 공유**
    
    - 팀별/개인별 프로젝트 시연
        
    - GitHub Repo 및 실행 로그 화면 캡처
        
    - 보고서 작성 및 제출
        
---

## 2️⃣ 사전 준비 점검

### (1) GitHub Secrets 설정 여부

- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`
    

### (2) Terraform Outputs

📄 **terraform/outputs.tf**

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
### (3) Ansible Inventory

📄 **ansible/inventory.ini**

```ini
[webserver]
{{ ec2_public_ip }} ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

### (4) Playbook 변수 전달

📄 **ansible/playbook.yml**

```yaml
- name: Deploy Flask Webserver
  hosts: webserver
  become: true
  vars:
    rds_endpoint: "{{ rds_endpoint }}"
    db_user: "admin"
    db_password: "Passw0rd123!"
    db_name: "sajuapp"
  roles:
    - web
```

---
<!-- _class: small -->

## 3️⃣ 최종 GitHub Workflow

📄 **.github/workflows/deploy.yml**

```yaml
name: CI/CD Final Project

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init
        working-directory: ./terraform

      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./terraform

      - name: Get Terraform Outputs
        id: tf-out
        run: |
          cd terraform
          echo "ec2_public_ip=$(terraform output -raw ec2_public_ip)" >> $GITHUB_ENV
          echo "rds_endpoint=$(terraform output -raw rds_endpoint)" >> $GITHUB_ENV
          echo "alb_dns_name=$(terraform output -raw alb_dns_name)" >> $GITHUB_ENV

      - name: Install Ansible
        run: sudo apt-get update && sudo apt-get install -y ansible

      - name: Run Ansible Playbook
        run: |
          ansible-playbook -i ansible/inventory.ini ansible/playbook.yml \
            --extra-vars "rds_endpoint=${{ env.rds_endpoint }} ec2_public_ip=${{ env.ec2_public_ip }}"
```

---

## 4️⃣ 최종 시연 절차

1. **코드 수정**  
    📄 `app.py`
    
    ```python
    @app.route("/")
    def hello():
        return "🎉 최종 프로젝트 – CI/CD 자동 배포 성공!"
    ```
    
2. **커밋 & 푸시**
    
    ```bash
    git add app.py
    git commit -m "Update final project message"
    git push origin main
    ```

---
3. **GitHub Actions 실행 확인**
    
    - GitHub Repo → Actions 탭 → Workflow 로그 확인
        
    - Terraform → Ansible 순서대로 실행되는지 확인
        
4. **ALB DNS 접속**
    
    ```
    http://<ALB_DNS>
    ```
    
    → 수정된 메시지가 표시되는지 확인
    

---

## 5️⃣ 발표 & 제출물

- **제출물**
    
    - GitHub Repo (코드, 워크플로우, README 포함)
        
    - 최종 보고서 (설계, 실행 결과 캡처, 문제 해결 과정)
        
    - 발표 자료 (PPT, 시연 영상 가능)
        
- **발표 구성**
    
    1. 프로젝트 개요 (Flask + Terraform + Ansible + CI/CD)
        
    2. 인프라 아키텍처 다이어그램
        
    3. GitHub Actions Workflow 설명
        
    4. 실행 시연 (ALB 접속 화면)
        
    5. 문제 해결 경험 공유 (트러블슈팅 사례)
        

---

## 6️⃣ 학습 정리

- **한 학기 종합 성과**
    
    - Flask 앱 개발
        
    - Terraform → 인프라 자동화
        
    - Ansible → 서버 내부 자동화
        
    - GitHub Actions → CI/CD 파이프라인 완성
        
- **학생 성과물**
    
    - GitHub 포트폴리오
        
    - AWS 기반 DevOps 실습 경험
        
    - 팀 발표 & 보고서 → 취업 면접에서도 활용 
        

---

## ✅ 과제 (최종)

1. ALB DNS 주소와 웹 결과 캡처 → 보고서에 포함
    
2. GitHub Actions 실행 로그 캡처 제출
    
3. 팀별/개인별 **프로젝트 발표 5분** 진행
    

---
## ✅  팁
GitHub Actions에서 워크플로우 YAML을 수정하고 다시 **커밋 + 푸시**하면 → Workflow는 **처음부터 다시 실행**
그런데 **이전에 실패한 리소스가 AWS에 남아있으면** 다시 실행할 때 또 충돌 에러가 납니다.

---

## 🔎 왜 이런 문제가 생기냐?

- Terraform은 `terraform.tfstate` 파일(상태파일)을 기준으로 AWS 리소스를 관리합니다.
    
- GitHub Actions에서는 매번 새로운 runner(임시 서버)에서 실행되기 때문에, **state 파일이 로컬에 남아 있지 않습니다.**
    
- 즉, 이전에 만든 리소스는 AWS에 그대로 남아있는데, Actions 쪽은 모른 채 “새로 만들겠다” → **중복 에러** 발생.
    

---

## ✅ 해결 방법

### 방법 1. 원격 State Backend 사용 (추천, 실무형)

- Terraform state를 **S3 + DynamoDB**에 저장해두면,  
    GitHub Actions도 항상 최신 상태를 공유할 수 있습니다.  
    예:
    

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "ap-northeast-2"
    dynamodb_table = "terraform-locks"
  }
}
```

👉 이렇게 하면, 워크플로우가 새로 실행돼도 state가 남아있어 **중복 생성 에러 없음**.


