
# 📘 클라우드 인프라 자동화와 CI/CD 
## Part 1. Terraform 인프라 자동화

## Part 2. Ansible 자동화 & CI/CD

---
## Part 1. Terraform 인프라 자동화
## 🎯 학습 목표

- 인프라를 **코드(IaC)** 로 관리하는 방법 이해
    
- AWS 기반 **네트워크 + 컴퓨팅 + 데이터베이스** 자동화
    
- **보안 그룹, 로드밸런서** 구성 실습
    
- 최종적으로 **간단한 웹을 Terraform으로 배포**

---
## 📌 Part 1.  강의 목차

- **1주차**: Terraform 기본 (Provider, Resource, Variable)
    
- **2주차**: VPC + Subnet + EC2 + Security Group 구성
    
- **3주차**: RDS(MySQL) 자동 배포 및 연결
    
- **4주차**: ALB 연결 → 외부 접속 가능하게 만들기
    
- **5~7주차**: 🚀 **Mini Project ② – Terraform으로 AI 사주 앱 배포**


---
## Part 2. Ansible 자동화 & CI/CD
## 🎯 학습 목표

- **Ansible**을 활용한 서버 구성 자동화 이해
    
- Flask 웹앱 + RDS DB를 **자동 배포**
    
- GitHub Actions로 **CI/CD 파이프라인 구축**
    
- Terraform, Ansible, GitHub Actions의 **연계 자동화 실습**
    
- 최종적으로 **AI 사주 앱 CI/CD 환경 완성**
---

## 📌 Part 2.  강의목차

- **9주차**: Ansible 기초 (Inventory, Playbook, Role 구조)
    
- **10주차**: Flask + DB 자동 배포 (Ansible)
    
- **11주차**: GitHub Actions 기본 구조 (Workflow, Event, Job, Step)
    
- **12주차**: GitHub Actions + Terraform + Ansible → CI/CD 자동화
    
- **13~14주차**: 🚀 **Mini Project ③ – AI 사주 앱 CI/CD 완성**

---

# 📘 Part 1. Terraform 인프라 자동화
---

  






# 📘 1주차 실습 가이드: Terraform 기본 – Provider, Resource, Variable

## 🎯 학습 목표

- Terraform의 기본 동작 원리를 이해한다.
    
- `provider`, `resource`, `variable` 개념과 문법을 익힌다.
    
- AWS EC2를 생성하는 간단한 IaC 코드를 작성하고 실행한다.
    
- `terraform init → plan → apply → destroy` 기본 워크플로우를 실습한다.
    

---

## 🌐 왜 Terraform인가?

- 멀티 클라우드 지원 (AWS, Azure, GCP 등)
    
- 선언형 IaC 언어 (HCL) → **재현성 + 협업 + 버전관리**
    
- DevOps / DevSecOps 환경에서 필수 도구

---
## 1️⃣ Terraform 개요

- **IaC(Infrastructure as Code)**: 인프라를 코드로 선언 → 버전관리 + 재현 가능.
    
- **Terraform 특징**
    
    - 멀티 클라우드 지원 (AWS, GCP, Azure 등).
        
    - 선언형 언어 (HCL, HashiCorp Configuration Language).
        
    - `terraform plan`으로 변경사항 확인, `terraform apply`로 실행.
        

---

## 2️⃣ 핵심 개념

### (1) Provider

- Terraform이 사용할 클라우드/플랫폼을 지정.  
    예: AWS, Azure, Google Cloud 등.
    

```hcl
provider "aws" {
  region = "ap-northeast-2"   # 서울 리전
}
```
---
### (2) Resource

- 실제 생성되는 인프라 객체.  
    예: EC2, S3, VPC, RDS 등.
    

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c9c942bd7bf113a2" # Ubuntu AMI (서울)
  instance_type = "t2.micro"
}
```
---
### (3) Variable

- 코드 내에서 **재사용 가능한 값**을 정의.
    

```hcl
variable "region" {
  default = "ap-northeast-2"
}

variable "instance_type" {
  default = "t2.micro"
}
```

→ 코드 내에서 `${var.region}`, `${var.instance_type}` 으로 참조.

---
## 3️⃣ 실습 예제

**파일 구조**
```
terraform-basics/
 ├─ main.tf
 ├─ variables.tf
 └─ terraform.tfvars
```
**main.tf**
```hcl
provider "aws" {
  region = var.region
}
resource "aws_instance" "web" {
  ami           = "ami-0c9c942bd7bf113a2" # Ubuntu 20.04 LTS
  instance_type = var.instance_type
  tags = {
    Name = "terraform-ec2"
  }
}
```
---
**variables.tf**

```hcl
variable "region" {
  description = "AWS region"
  default     = "ap-northeast-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}
```

**terraform.tfvars**

```hcl
region        = "ap-northeast-2"
instance_type = "t3.micro"
```

---

## 4️⃣ 실행 절차

```bash
# 1. 초기화 (플러그인 다운로드)
terraform init

# 2. 실행 계획 확인
terraform plan

# 3. 실제 적용 (EC2 생성)
terraform apply

# 4. 리소스 삭제
terraform destroy
```

---

## 5️⃣ 학습 정리

- Provider = 클라우드 선택 (AWS, GCP, Azure 등).
    
- Resource = 생성할 실제 인프라 (EC2, RDS, ALB 등).
    
- Variable = 값 재사용 (환경에 따라 쉽게 변경 가능).
    
- Terraform은 **init → plan → apply → destroy** 흐름으로 동작.
    
- **1주차 결과물**: `terraform apply` 후 AWS 콘솔에서 EC2 인스턴스 생성 확인.
    




---
