
# 📝 4주차 실습 가이드 – ALB 연결 및 외부 접속

## 🎯 실습 목표

- Application Load Balancer(ALB) 개념 이해
    
- Terraform으로 ALB, Target Group, Listener 생성
    
- EC2 인스턴스를 Target Group에 등록
    
- 브라우저에서 ALB DNS를 통해 외부 접속 확인
    

---

## 📌 1. 아키텍처 개요

이번 주차에서 완성할 구조:

```
[VPC]
 ├── [Public Subnet] → EC2 (Flask App)
 ├── [Private Subnet] → RDS (MySQL)
 └── [ALB] → Target Group → EC2
```

---
<!-- _class: medium -->

## 📌 2. 코드 작성
### (1) ALB Security Group
```hcl
resource "aws_security_group" "alb_sg" {
  vpc_id = aws_vpc.main.id
  name   = "terraform-alb-sg"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "terraform-alb-sg"
  }
}
```
---
### (2) Application Load Balancer

```hcl
resource "aws_lb" "app_alb" {
  name               = "terraform-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public.id] # 퍼블릭 서브넷

  tags = {
    Name = "terraform-alb"
  }
}
```
---
### (3) Target Group

```hcl
resource "aws_lb_target_group" "app_tg" {
  name     = "terraform-tg"
  port     = 5000
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }

  tags = {
    Name = "terraform-tg"
  }
}
```
---
### (4) Target Group Attachment

```hcl
resource "aws_lb_target_group_attachment" "app_attach" {
  target_group_arn = aws_lb_target_group.app_tg.arn
  target_id        = aws_instance.web.id
  port             = 5000
}
```

### (5) Listener

```hcl
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

## 📌 3. Terraform 실행

```bash
terraform init
terraform plan
terraform apply
```

---

## 📌 4. 확인 방법

1. **AWS Console → EC2 → Load Balancers**
    
    - `terraform-alb` 상태 확인 (Active)
        
    - DNS 이름 복사
        
2. **브라우저 접속**
    
    ```
    http://<ALB-DNS>
    ```
    
    - 현재는 Flask 앱이 단순히 실행만 되고 있다면, “Hello World” 또는 빈 응답 확인
        

---

## 📌 5. 정리

- **이번 주차 핵심**:
    
    - ALB → EC2 라우팅 구조 이해
        
    - Security Group: ALB(80) → EC2(5000) 연결 체인 완성
        
- **결과물**:
    
    - ALB DNS로 접근 시, EC2에서 동작하는 Flask 앱 응답 확인
        

---

## ✅ 과제

1. Flask 앱 실행 포트를 `8080`으로 바꾸고 Target Group 포트도 `8080`으로 수정 후 다시 `apply`
    
2. Target Group Health Check 경로를 `/health`로 바꿔보고 동작 확인
    
3. EC2 인스턴스를 2대 만들고 Target Group에 모두 등록 → 로드밸런싱 테스트
    

---
ALB(Application Load Balancer)는 **반드시 2개 이상의 Subnet, 서로 다른 AZ**에 걸쳐 있어야 생성됩니다.  
지금 코드에서는 `subnets = [aws_subnet.public.id]` 이렇게 하나만 넣으셨을 겁니다 → 그래서 AWS가 거부한 거예요.

---
## 🛠 해결 방법 Subnet 2개 만들기 (AZ 분산)
```hcl
# Public Subnet 1 (ap-northeast-2a)
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "terraform-public-a"
  }
}
# Public Subnet 2 (ap-northeast-2b)
resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.3.0/24"
  availability_zone       = "ap-northeast-2b"
  map_public_ip_on_launch = true

  tags = {
    Name = "terraform-public-b"
  }
}
```
---
### 2. ALB에 두 서브넷 연결

```hcl
resource "aws_lb" "app_alb" {
  name               = "terraform-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id] # 두 개 필수

  tags = {
    Name = "terraform-alb"
  }
}
```

---

## 📌 정리

- ALB는 “고가용성”이 목적 → 반드시 **2개 이상의 AZ 서브넷** 필요.
    
- 실습에서는 `10.0.1.0/24`(2a), `10.0.3.0/24`(2b) 이런 식으로 CIDR만 다르게 두 개 잡으면 됩니다.
    

---
## 🔎 에러 원인

```
Reference to undeclared resource
aws_subnet.public.id
```

Terraform이 말하는 건 **`aws_subnet.public`이라는 리소스는 코드 안에 없는데 참조하고 있다**는 거예요.

- 교수님이 ALB 붙이면서 `aws_subnet.public`을 삭제(또는 이름 변경)했는데,
    
- EC2, Route Table Association, RDS Subnet Group 등에서 여전히 `aws_subnet.public`을 바라보고 있어서 생긴 겁니다.
    

---

## 🛠 해결 방법

### 1. Subnet 리소스 이름 정리

제가 직전 답변에서 Subnet을 `public_a`, `public_b`로 나눠드렸죠?  
그럼 이제 코드 전체에서 `aws_subnet.public` 대신 → `aws_subnet.public_a` 또는 `aws_subnet.public_b` 로 바꿔줘야 합니다.

예시:

**수정 전**

```hcl
subnet_id = aws_subnet.public.id
```

**수정 후 (EC2는 2a에만 배치한다고 가정)**

```hcl
subnet_id = aws_subnet.public_a.id
```
---
**RDS Subnet Group (퍼블릭+프라이빗 대신 새 네트워크 사용)**

```hcl
subnet_ids = [aws_subnet.public_a.id, aws_subnet.public_b.id]
```

**Route Table Association**

```hcl
subnet_id      = aws_subnet.public_a.id
route_table_id = aws_route_table.public.id
```

---

### 2. 네이밍 일관성 유지

- `aws_subnet.public` → 더 이상 없음
    
- 새로 만든 건 `aws_subnet.public_a`, `aws_subnet.public_b`, `aws_subnet.private`
    

즉, main.tf 전체에서 **public → public_a 또는 public_b**로 교체해야 합니다.

---

## 📌 정리

- 지금 에러는 리소스 이름 충돌이 아니라 **리소스 참조 불일치**
    
- 해결책은 “`public` 없앴으면 → 참조하는 곳 전부 `public_a` or `public_b`로 맞추기”
    

---



아래는 **5주차 기준 main.tf 완성본**입니다.  
구조는 VPC → Subnet 3개(퍼블릭 2, 프라이빗 1) → IGW → Route Table → SG(웹/DB/ALB) → EC2 → RDS → ALB 순서로 정리했습니다.

---
<!-- _class: medium -->

# 📄 main.tf (5주차 완성본)

```hcl
provider "aws" {
  region = "ap-northeast-2"
}
# -----------------------------
# VPC
# -----------------------------
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "terraform-vpc"
  }
}
# -----------------------------
# Subnets (2 Public, 1 Private)
# -----------------------------
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.11.0/24"
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true
  tags = { Name = "public-a" }
}
```
---
```hcl
resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.12.0/24"
  availability_zone       = "ap-northeast-2b"
  map_public_ip_on_launch = true
  tags = { Name = "public-b" }
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.21.0/24"
  availability_zone = "ap-northeast-2c"
  tags = { Name = "private" }
}

# -----------------------------
# Internet Gateway & Route Table
# -----------------------------
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "terraform-igw" }
}
```
---
```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
  tags = { Name = "public-rt" }
}

resource "aws_route_table_association" "public_a_assoc" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_b_assoc" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public.id
}
```
---
```hcl
# -----------------------------
# Security Groups
# -----------------------------
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id
  name   = "web-sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 5000
    to_port     = 5000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "web-sg" }
}
```
---
```hcl
resource "aws_security_group" "db_sg" {
  vpc_id = aws_vpc.main.id
  name   = "db-sg"

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web_sg.id] # 웹 SG에서만 허용
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "db-sg" }
}
```
---
```hcl
resource "aws_security_group" "alb_sg" {
  vpc_id = aws_vpc.main.id
  name   = "alb-sg"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "alb-sg" }
}
```
---
```hcl
# -----------------------------
# EC2 (Flask App 서버)
# -----------------------------
resource "aws_instance" "web" {
  ami                         = "ami-0c9c942bd7bf113a2" # Ubuntu 20.04 LTS
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public_a.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  tags = { Name = "terraform-ec2" }
}
```
---
```hcl
# -----------------------------
# RDS (MySQL)
# -----------------------------
resource "aws_db_subnet_group" "db_subnet" {
  name       = "db-subnet"
  subnet_ids = [aws_subnet.private.id, aws_subnet.public_b.id]
  tags       = { Name = "db-subnet" }
}
resource "aws_db_instance" "mysql" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  db_name              = "sajuapp"
  username             = "admin"
  password             = "Passw0rd123!"
  parameter_group_name = "default.mysql8.0"
  skip_final_snapshot  = true

  vpc_security_group_ids = [aws_security_group.db_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.db_subnet.name

  tags = { Name = "terraform-rds" }
}
```
---
```hcl
# -----------------------------
# ALB
# -----------------------------
resource "aws_lb" "app_alb" {
  name               = "terraform-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id]

  tags = { Name = "terraform-alb" }
}

resource "aws_lb_target_group" "app_tg" {
  name     = "terraform-tg"
  port     = 5000
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }

  tags = { Name = "terraform-tg" }
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

## ✅ 실행 순서

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

- ALB 생성 후, AWS Console → Load Balancer → DNS 이름 복사
    
- 브라우저에서 접속 → EC2의 5000 포트 Flask 앱 응답 확인
    

---


# 📝 ec2 SSH 접속
 
 **Terraform으로 Key Pair 생성 → EC2에 연결**까지 자동화하는 코드를 추가

---
# 📄 main.tf (Key Pair + EC2 접속 가능 설정)

```hcl
# -----------------------------
# Key Pair (EC2 SSH 접속용)
# -----------------------------
resource "tls_private_key" "ssh_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "deployer" {
  key_name   = "terraform-key"
  public_key = tls_private_key.ssh_key.public_key_openssh
}

# 로컬에 프라이빗 키 저장
resource "local_file" "private_key_pem" {
  content  = tls_private_key.ssh_key.private_key_pem
  filename = "${path.module}/terraform-key.pem"
  file_permission = "0600"
}
```

---
```hcl
# -----------------------------
# EC2 (Flask App 서버)
# -----------------------------
resource "aws_instance" "web" {
  ami                         = "ami-0c9c942bd7bf113a2" # Ubuntu 20.04 LTS
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public_a.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  key_name = aws_key_pair.deployer.key_name  # Key Pair 연결

  tags = { Name = "terraform-ec2" }
}
```

---

## ✅ 실행 절차

```bash
terraform init
terraform apply -auto-approve
```

1. 실행하면 `terraform-key.pem` 파일이 **main.tf가 있는 디렉토리**에 자동 생성됩니다.  
    (권한은 0600으로 저장됨 → 바로 SSH 접속 가능)
    
2. EC2 생성 후, 접속 명령어는 다음과 같습니다:
    
    ```bash
    ssh -i terraform-key.pem ubuntu@<EC2_PUBLIC_IP>
    ```
    

---

## 📌 정리

- 이제 Key Pair 생성과 관리가 Terraform 안에서 완전 자동화됨.
    
- 매번 콘솔에서 `.pem`을 다운받을 필요 없음.
    
- EC2는 항상 `terraform-key`라는 Key Pair를 사용.
    
- 접속 키는 로컬에 `terraform-key.pem`으로 저장되므로 바로 SSH 가능.
    

---

# 📝ec2 ssh 키 키 저장 위치

---

## 1️⃣  코드

```hcl
resource "local_file" "private_key_pem" {
  content  = tls_private_key.ssh_key.private_key_pem
  filename = "${path.module}/terraform-key.pem"
  file_permission = "0600"
}
```

- `${path.module}` = **main.tf가 있는 디렉토리 경로**
    
- 그래서 실행할 때마다 **Terraform 프로젝트 폴더(예: ~/politec-book/)** 밑에 `terraform-key.pem`이 생기는 겁니다.
    

---

## 2️⃣ 우리가 흔히 쓰는 SSH 키 위치

- 리눅스/맥에서 기본 경로는 `~/.ssh/`
    
- AWS 콘솔에서 `.pem` 다운로드하면 보통 교수님이 직접 `~/.ssh/`로 옮겨놓고 씁니다.
    
- Terraform은 기본적으로 거기다 저장하지 않고, 코드에서 지정한 곳에 저장합니다.
    

---

## 3️⃣ 해결책

원하는 위치를 코드에서 직접 지정하면 됩니다.

예시: `~/.ssh/terraform-key.pem`에 저장하고 싶다면:

```hcl
resource "local_file" "private_key_pem" {
  content  = tls_private_key.ssh_key.private_key_pem
  filename = "${pathexpand("~/.ssh/terraform-key.pem")}"
  file_permission = "0600"
}
```

---

## ✅ 정리

- 지금은 `${path.module}` 때문에 **프로젝트 폴더(main.tf 옆)**에 생김
    
- `~/.ssh/` 밑에 만들고 싶으면 `pathexpand("~/.ssh/terraform-key.pem")`로 경로를 바꾸면 됨
    

---
