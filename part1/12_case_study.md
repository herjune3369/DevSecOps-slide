# 📝 12주차 실습 가이드 – GitHub Actions + Terraform Output → Ansible 자동 전달

---

## 🎯 학습 목표

- GitHub Actions에서 Terraform을 실행해 인프라 배포
    
- Terraform output 값(RDS Endpoint, EC2 Public IP 등)을 Ansible에 자동 전달
    
- Ansible이 output을 받아 Flask 앱을 자동 배포
    
- **코드 한 번 push → 인프라+앱 자동화** 전체 흐름 완성
    

---

## 1️⃣ Terraform Output 정의

📂 **terraform/outputs.tf**

```hcl
output "ec2_public_ip" {
  value = aws_instance.web.public_ip
}

output "rds_endpoint" {
  value = aws_db_instance.mysql.endpoint
}
```

👉 이 파일이 있어야 GitHub Actions에서 output을 추출 가능

---
<!-- _class: medium -->

## 2️⃣ GitHub Actions Workflow 수정

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

      # 4. Terraform Output 추출
      - name: Get Terraform Outputs
        id: tf-out
        run: |
          cd terraform
          echo "ec2_public_ip=$(terraform output -raw ec2_public_ip)" >> $GITHUB_ENV
          echo "rds_endpoint=$(terraform output -raw rds_endpoint)" >> $GITHUB_ENV
```
---
    ```yaml
	  # 5. Ansible 실행
      - name: Install Ansible
        run: sudo apt-get update && sudo apt-get install -y ansible

      - name: Run Ansible Playbook
        run: |
          ansible-playbook -i ansible/inventory.ini ansible/playbook.yml \
            --extra-vars "rds_endpoint=${{ env.rds_endpoint }} ec2_public_ip=${{ env.ec2_public_ip }}"
            ```
            
            



---

## 3️⃣ Ansible Inventory 수정

📄 **ansible/inventory.ini**

```ini
[webserver]
{{ ec2_public_ip }} ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

👉 여기서 `{{ ec2_public_ip }}`는 GitHub Actions에서 전달받은 변수로 치환

---

## 4️⃣ Ansible Playbook 수정

📄 **ansible/playbook.yml**

```yaml
- name: Deploy Flask Webserver with DB Connection
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

👉 여기서 `rds_endpoint`는 GitHub Actions에서 Terraform output으로 전달됨

---

## 5️⃣ 실행 시나리오

1. 학생이 `app.py` 코드 수정 후 GitHub에 push
    
2. GitHub Actions 실행
    
    - Terraform → EC2, RDS 배포
        
    - Output 추출 (EC2 IP, RDS Endpoint)
        
    - Ansible 실행 시 → output 값 자동 전달
        
    - Flask 앱이 새로 배포
        
3. 브라우저에서 ALB DNS 접속 → 새로운 앱 확인
    

---

## 6️⃣ 학습 정리

- **Terraform Output**: “Terraform이 만든 자원을 Ansible이 알아서 쓴다”
    
- **GitHub Actions**: Output을 받아 Ansible에 변수로 전달
    
- 학생들은 **코드 → GitHub → 자동 배포**의 전체 DevOps 파이프라인 완성을 체험
    

---

## ✅ 과제

1. Terraform Output에 `alb_dns_name` 추가 → Actions 로그에 출력
    
2. Ansible 템플릿에서 `alb_dns_name`을 Flask `/alb` 라우트에서 보여주도록 수정
    
3. GitHub Actions 실행 후, 변경된 웹페이지와 ALB 로그 스크린샷 제출
    

