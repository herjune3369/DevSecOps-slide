
# 3주차 실습 가이드 – RDS(MySQL) 자동 배포 및 연결

## 🎯 실습 목표

- Terraform으로 RDS(MySQL) 인스턴스를 생성한다.
    
- VPC/Subnet/SG에 맞게 DB를 배치하고 외부 접속을 제어한다.
    
- EC2 인스턴스에서 MySQL 클라이언트로 DB 접속을 확인한다.
    

---

## 📌 1. 아키텍처 개요

이번 주차에서 완성할 구조:

```
[VPC]
 ├── [Public Subnet] → EC2 (App 서버)
 └── [Private Subnet] → RDS (MySQL)
       └── Security Group (EC2에서만 접속 허용)
```

---

Internet
    ↓
Internet Gateway
    ↓
Public Subnet (10.0.1.0/24) - EC2 Instance 
    ↓
Private Subnet (10.0.2.0/24) - RDS MySQL Database

---
## 📌 2. 디렉토리 준비

7주차 디렉토리(`terraform-vpc`)를 그대로 사용하거나, 새로운 `terraform-rds` 디렉토리를 만들어 확장합니다.

```bash
mkdir ~/terraform-rds
cd ~/terraform-rds
```

---
## 📌 3. 코드 작성

### (1) VPC/EC2 코드 재사용

7주차 `main.tf` 내용을 그대로 유지.  
추가로 RDS 관련 코드만 붙입니다.

### (2) DB Subnet Group

```hcl
# DB Subnet (Private)
resource "aws_subnet" "private" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "ap-northeast-2b"

  tags = {
    Name = "terraform-private-subnet"
  }
}
```

---
```hcl
# DB Subnet Group (RDS 전용)
resource "aws_db_subnet_group" "db_subnet" {
  name       = "terraform-db-subnet"
  subnet_ids = [aws_subnet.public.id, aws_subnet.private.id]

  tags = {
    Name = "terraform-db-subnet"
  }
}
```
---
### (3) DB Security Group

```hcl
resource "aws_security_group" "db_sg" {
  vpc_id = aws_vpc.main.id
  name   = "terraform-db-sg"
  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web_sg.id] # EC2 SG에서만 허용
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "terraform-db-sg"
  }
}
```

---
### (4) RDS(MySQL) 리소스

```hcl
resource "aws_db_instance" "mysql" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  db_name              = "sajuapp"
  username             = var.db_username
  password             = var.db_password
  parameter_group_name = "default.mysql8.0"
  skip_final_snapshot  = true

  vpc_security_group_ids = [aws_security_group.db_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.db_subnet.name

  tags = {
    Name = "terraform-rds-mysql"
  }
}
```

---

## 📌 4. 변수 정의

**variables.tf**

```hcl
variable "region" {
  default     = "ap-northeast-2"
  description = "AWS region"
}

variable "instance_type" {
  default     = "t2.micro"
  description = "EC2 instance type"
}

variable "db_username" {
  description = "Database admin username"
  default     = "admin"
}

variable "db_password" {
  description = "Database admin password"
  sensitive   = true
}
```
---
**terraform.tfvars**

```hcl
region        = "ap-northeast-2"
instance_type = "t3.micro"
db_username   = "admin"
db_password   = "Passw0rd123!"
```

---

## 📌 5. 실행 절차

```bash
terraform init
terraform plan
terraform apply
```

---

## 📌 6. 확인 방법

1. **AWS Console → RDS**
    
    - `terraform-rds-mysql` 인스턴스 확인
        
    - Subnet Group, Security Group 확인
        
2. **EC2 접속 후 MySQL 클라이언트 설치**
    
    ```bash
    sudo apt-get update && sudo apt-get install -y mysql-client
    mysql -h <RDS-ENDPOINT> -u admin -p
    ```
    
    → RDS 엔드포인트는 Terraform 출력이나 콘솔에서 확인
    

---

## 📌 7. 정리

- **이번 주차 핵심**:
    
    - RDS는 반드시 **DB Subnet Group** 안에 배치해야 함.
        
    - Security Group으로 **EC2 → RDS만 허용**하는 구조 설계.
        
- **결과물**:
    
    - App 서버(EC2) + DB 서버(RDS) 완성
        
    - 다음 주차(9주차)에서 **ALB 연결**까지 확장
        

---

## ✅ 과제

1. DB 이름(`sajuapp`)을 `studentdb`로 변경 후 다시 apply → 반영되는지 확인
    
2. RDS 인스턴스 타입을 `db.t3.small`로 변경 후 비용 차이 확인
    
3. Terraform `output` 블록을 추가해서 RDS Endpoint 자동 출력하도록 수정
    

---
