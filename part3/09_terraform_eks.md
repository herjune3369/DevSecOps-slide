# 9주차 이론 강의 – Terraform 기본 문법과 클라우드 인프라(EKS 등) 배포

## 1. IaC(Infrastructure as Code) 개념

- 인프라를 수동으로 콘솔에서 클릭하지 않고 **코드로 선언**
    
- 버전 관리, 재현성, 자동화 가능
    
- DevOps 파이프라인에 자연스럽게 통합
    

---

## 2. Terraform 개요

- HashiCorp에서 개발한 오픈소스 IaC 도구
    
- 선언형 언어(HCL, HashiCorp Configuration Language) 사용
    
- **실행 흐름**
    
    1. 코드 작성 (`.tf` 파일)
        
    2. `terraform init` → 플러그인 다운로드
        
    3. `terraform plan` → 변경사항 미리보기
        
    4. `terraform apply` → 실제 리소스 생성
        
    5. `terraform destroy` → 리소스 삭제
        

---

## 3. Terraform 기본 문법

### Provider

- 리소스를 생성할 클라우드/플랫폼 지정
    

```hcl
provider "aws" {
  region = "ap-northeast-2"
}
```

### Resource

- 생성할 실제 인프라 정의
    

```hcl
resource "aws_instance" "demo" {
  ami           = "ami-0c9c942bd7bf113a2" # Ubuntu 22.04
  instance_type = "t2.micro"
  tags = {
    Name = "terraform-demo"
  }
}
```
---
### Variable

- 코드에서 반복되는 값을 변수로 관리
    

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

### Output

- 생성된 리소스 속성을 출력
    

```hcl
output "public_ip" {
  value = aws_instance.demo.public_ip
}
```

---

## 4. State 파일

- Terraform은 현재 리소스 상태를 `terraform.tfstate` 파일에 저장
    
- 협업 시 **원격 백엔드(S3, DynamoDB)**를 사용해 상태 공유
    

---

## 5. AWS EKS 배포 개요

- EKS = AWS 관리형 Kubernetes 클러스터
    
- Terraform을 사용하면 VPC, Subnet, Node Group까지 자동 구성 가능
    
- 주요 리소스:
    
    - VPC, Subnet, Internet Gateway
        
    - EKS 클러스터(Control Plane)
        
    - EKS Node Group (Worker Node)
        

---

## 6. EKS 배포 흐름 (Terraform)

1. VPC와 Subnet 생성 (Public, Private 분리)
    
2. EKS 클러스터 정의
    
3. Node Group 추가 (EC2 기반 Worker Node)
    
4. `aws eks update-kubeconfig`로 kubeconfig 갱신
    
5. `kubectl get nodes`로 연결 확인
    

---

## 7. Terraform 장점

- 멀티 클라우드 지원 (AWS, GCP, Azure 등)
    
- 선언형 관리 (YAML과 유사, 원하는 상태만 정의)
    
- DevOps 파이프라인 통합 (CI/CD 자동화와 결합)
    
- 실무에서 EKS 같은 복잡한 인프라도 **한 번의 코드 실행으로 배포** 가능
    

---

## ✅ 정리

- Terraform = 인프라를 코드로 선언하고 자동 관리하는 도구
    
- Provider, Resource, Variable, Output이 핵심 문법
    
- AWS EKS 클러스터도 Terraform 코드 몇 줄로 생성 가능
    
- 다음(10주차)에서는 Terraform으로 만든 Kubernetes 클러스터 위에 **Ansible을 활용한 앱 배포 자동화**를 학습
    

---



## 🎯 실습 목표

- Terraform 설치와 기본 명령어를 익힌다.
    
- Provider, Resource, Variable, Output을 코드로 작성한다.
    
- AWS에서 EC2 인스턴스를 생성하고 삭제한다.
    
- Terraform으로 EKS 클러스터를 배포하고 `kubectl`로 연결한다.
    

---

## 1. 환경 준비

### 1-1. AWS CLI 설정

```bash
aws configure
```

- `AWS Access Key`, `Secret Key` 입력
    
- Region: `ap-northeast-2` (서울)
    
- Output: `json`
    

### 1-2. Terraform 설치 확인

```bash
terraform -version
```

✅ `Terraform v1.x.x` 출력

---

## 2. 프로젝트 디렉토리 생성

```bash
mkdir ~/terraform-eks-demo
cd ~/terraform-eks-demo
```

---

## 3. EC2 생성 실습

### 3-1. Provider (`provider.tf`)

```hcl
provider "aws" {
  region = "ap-northeast-2"
}
```

### 3-2. EC2 리소스 (`main.tf`)

```hcl
resource "aws_instance" "demo" {
  ami           = "ami-0c9c942bd7bf113a2" # Ubuntu 22.04 LTS (서울 리전)
  instance_type = "t2.micro"
  tags = {
    Name = "terraform-demo"
  }
}
```
---
### 3-3. Output (`outputs.tf`)

```hcl
output "instance_ip" {
  value = aws_instance.demo.public_ip
}
```

### 3-4. 실행

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

✅ EC2 인스턴스 생성, 퍼블릭 IP 출력

---
### 3-5. 접속 확인

```bash
ssh -i <your-key.pem> ubuntu@<instance_ip>
```

### 3-6. 삭제

```bash
terraform destroy -auto-approve
```

---

## 4. EKS 클러스터 배포

### 4-1. VPC 및 Subnet (`vpc.tf`)

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-northeast-2a", "ap-northeast-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
}
```

---
### 4-2. EKS 클러스터 (`eks.tf`)

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.3"

  cluster_name    = "eks-demo"
  cluster_version = "1.29"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    demo = {
      desired_size = 2
      max_size     = 3
      min_size     = 1
      instance_types = ["t3.medium"]
    }
  }
}
```

---
### 4-3. Outputs (`outputs.tf`)

```hcl
output "cluster_name" {
  value = module.eks.cluster_name
}
```

### 4-4. 실행

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

✅ VPC, Subnet, EKS 클러스터 생성

---

## 5. kubectl 연결

### 5-1. kubeconfig 업데이트

```bash
aws eks update-kubeconfig --name eks-demo --region ap-northeast-2
```

### 5-2. 노드 확인

```bash
kubectl get nodes
```

✅ Worker Node 2개 `Ready` 상태 확인

---

## 6. 정리

- EC2 인스턴스 생성/삭제 실습으로 Terraform 기본기 학습
    
- EKS 클러스터까지 배포해 Kubernetes 환경 구축 완료
    

```bash
terraform destroy -auto-approve
```

---

## ✅ 최종 결과

- Terraform 기본 문법(Provider, Resource, Variable, Output) 체험
    
- EC2 인스턴스 생성 및 삭제 성공
    
- EKS 클러스터 배포 후 `kubectl`로 연결 확인
    

