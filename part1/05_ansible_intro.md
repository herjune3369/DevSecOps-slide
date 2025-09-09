
# 5주차 실습 가이드 – Mini Project ②

Terraform으로 AI 사주 앱 배포

## 🎯 실습 목표

- Terraform을 이용해 **전체 인프라(네트워크, 서버, DB, 로드밸런서)**를 자동 구축
    
- EC2에 Flask AI 사주 앱 배포 (단순 버전)
    
- ALB DNS로 접속해 브라우저에서 앱 실행 확인
    

---

## 📌 1. 아키텍처 개요

```
[VPC]
 ├── [Public Subnet A] → EC2 (Flask App)
 ├── [Public Subnet B] → ALB (로드밸런서)
 └── [Private Subnet]  → RDS (MySQL)
```

- ALB DNS → EC2 (Flask App) → RDS(MySQL)
    

---

## 📌 2. Terraform 코드 구조

```
terraform-app/
 ├─ main.tf              # 인프라 정의 (VPC, EC2, RDS, ALB)
 ├─ variables.tf         # 변수 정의
 ├─ terraform.tfvars     # 실제 값
 └─ outputs.tf           # ALB DNS, RDS Endpoint 출력
```

---

## 📌 3. main.tf 핵심 블록 요약

### (1) Key Pair

```hcl
resource "tls_private_key" "ssh_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "deployer" {
  key_name   = "terraform-key"
  public_key = tls_private_key.ssh_key.public_key_openssh
}

resource "local_file" "private_key_pem" {
  content        = tls_private_key.ssh_key.private_key_pem
  filename       = pathexpand("~/.ssh/terraform-key.pem")
  file_permission = "0600"
}
```
---
### (2) EC2 (Flask App 서버)

```hcl
resource "aws_instance" "web" {
  ami                    = "ami-0c9c942bd7bf113a2"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public_a.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  key_name               = aws_key_pair.deployer.key_name
```
---
```hcl
  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get install -y python3-pip
              pip3 install flask
              cat << APP > /home/ubuntu/app.py
              from flask import Flask
              app = Flask(__name__)
              @app.route("/")
              def hello():
                  return "AI 사주 앱 – Terraform Mini Project!"
              if __name__ == "__main__":
                  app.run(host="0.0.0.0", port=5000)
              APP
              nohup python3 /home/ubuntu/app.py &
              EOF

  tags = { Name = "terraform-ec2" }
}
```
---

### (3) RDS (MySQL)

```hcl
resource "aws_db_instance" "mysql" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  db_name              = "sajuapp"
  username             = var.db_username
  password             = var.db_password
  skip_final_snapshot  = true

  vpc_security_group_ids = [aws_security_group.db_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.db_subnet.name
}
```
---
### (4) ALB

```hcl
resource "aws_lb" "app_alb" {
  name               = "terraform-alb"
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id]
}

resource "aws_lb_target_group" "app_tg" {
  name     = "terraform-tg"
  port     = 5000
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}
```
---
```hcl
resource "aws_lb_target_group_attachment" "app_attach" {
  target_group_arn = aws_lb_target_group.app_tg.arn
  target_id        = aws_instance.web.id
  port             = 5000
}

resource "aws_lb_listener" "app_listener" {
  load_balancer_arn = aws_lb.app_alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app_tg.arn
  }
}
```

---

## 📌 4. outputs.tf

```hcl
output "alb_dns_name" {
  description = "ALB DNS Name"
  value       = aws_lb.app_alb.dns_name
}

output "rds_endpoint" {
  description = "RDS Endpoint"
  value       = aws_db_instance.mysql.endpoint
}
```

---

## 📌 5. 실행 절차

```bash
terraform init
terraform apply -auto-approve
```

- ALB DNS 출력 확인:
    
    ```bash
    terraform output alb_dns_name
    ```
    
- 브라우저 접속:
    
    ```
    http://<ALB_DNS>
    ```
    
- 결과: “AI 사주 앱 – Terraform Mini Project!” 문자열 확인
    

---

## 📌 6. 학습 정리

- Terraform으로 **엔드투엔드 인프라(App + DB + LB)** 자동 배포 성공
    
- ALB DNS를 통해 외부 사용자도 접근 가능
    
- **Mini Project ② 완성** → 포트폴리오에 넣을 수 있는 클라우드 인프라 예제 확보
    

---

## ✅ 과제

1. Flask 앱에 `/saju` 라우트 추가해서 “오늘의 운세” 텍스트 출력
    
2. ALB Health Check 경로를 `/saju`로 변경
    
3. RDS 연결 부분(예: MySQL 접속 테스트 코드) 추가


---
