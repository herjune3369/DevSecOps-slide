#  2주차 실습 가이드– VPC, Subnet, EC2, Security Group

## 🎯 실습 목표

- VPC, Subnet, Internet Gateway, Route Table의 개념을 이해한다.
    
- Terraform으로 **네트워크 인프라**를 직접 정의한다.
    
- EC2 인스턴스를 새로 만든 VPC/서브넷 안에 배포한다.
    
- Security Group을 적용하여 외부 접속을 제어한다.
    

---

## 📌 1. 아키텍처 개요

이번 주차에서 구축할 구조:

```
[VPC]
 └── [Public Subnet]
       └── [EC2 Instance] ← (SSH 22, HTTP 80 허용)
       └── [Internet Gateway] + [Route Table]
```


## 📌 2. 디렉토리 준비

```bash
mkdir ~/terraform-vpc
cd ~/terraform-vpc
```

---
<!-- _class: medium -->

## 📌 3. 코드 작성

### (1) main.tf

```hcl
provider "aws" {
  region = var.region
}

# VPC 생성
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "terraform-vpc"
  }
}
# Subnet 생성
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "terraform-subnet"
  }
}
```
---

<!-- _class: medium -->

```hcl

# Internet Gateway
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = "terraform-igw"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "terraform-rt"
  }
}

# Route Table Association
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

```
---
<!-- _class: medium -->

```hcl


# Security Group
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id
  name   = "terraform-sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

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
    Name = "terraform-sg"
  }
}

```
---
```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami           = "ami-0c9c942bd7bf113a2" # Ubuntu 20.04 LTS (서울)
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  associate_public_ip_address = true

  tags = {
    Name = "terraform-ec2"
  }
}
```

### (3) terraform.tfvars

```hcl
region        = "ap-northeast-2"
instance_type = "t3.micro"
```

---

## 📌 4. 실행 절차

```bash
terraform init      # 플러그인 다운로드
terraform plan      # 실행 계획 확인
terraform apply     # 인프라 생성
```

---

## 📌 5. 확인 방법

1. **AWS 콘솔 → VPC**
    
    - 새 VPC(10.0.0.0/16), Subnet(10.0.1.0/24), IGW, Route Table 생성 확인
        
2. **AWS 콘솔 → EC2**
    
    - `terraform-ec2` 인스턴스 생성 확인
        
    - Public IP 확인 → 브라우저에서 접속 시 (현재는 Apache 미설치 → SSH 연결 테스트만 가능)
        

---

## 📌 6. 정리

- **이번 주차 핵심**:
    
    - Default VPC를 쓰지 않고, 직접 만든 VPC/서브넷에 EC2를 올린다.
        
    - 외부 접속을 위해 **IGW + Route Table + SG**를 반드시 구성해야 한다.
        
- **결과물**: Terraform 코드로 작성된 최소 네트워크 인프라 + EC2 인스턴스
    

---

## ✅ 과제

1. Security Group에 **포트 8080**을 추가하고 다시 `apply`
    
2. Subnet을 하나 더 만들어 `10.0.2.0/24` 구성 후, Route Table에 연결해보기
    

---
