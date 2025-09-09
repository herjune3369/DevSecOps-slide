
# 💻 15주차 실습 강의 – 인프라 자동화 (Terraform → EKS 배포)

## 🎯 실습 목표

- Terraform으로 AWS EKS 클러스터 자동 배포
    
- VPC + Subnet + NodeGroup까지 자동 생성
    
- `kubectl`을 통해 클러스터 정상 동작 확인
    

---

## 1. 환경 준비

### 1-1. AWS CLI 인증

```bash
aws configure
```

- `AWS_ACCESS_KEY_ID` 입력
    
- `AWS_SECRET_ACCESS_KEY` 입력
    
- `Default region`: `ap-northeast-2`
    
- `Output format`: `json`
    

### 1-2. Terraform 설치 확인

```bash
terraform -version
```

✅ `Terraform v1.x.x` 출력

---

## 2. 프로젝트 디렉토리 생성

```bash
mkdir ~/terraform-eks-project
cd ~/terraform-eks-project
```

---

<!-- _class: medium -->

## 3. Terraform 코드 작성

### 3-1. Provider (`provider.tf`)

```hcl
provider "aws" {
  region = "ap-northeast-2"
}
```

### 3-2. VPC 모듈 (`vpc.tf`)

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
<!-- _class: medium -->

### 3-3. EKS 모듈 (`eks.tf`)

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.3"

  cluster_name    = "eks-demo"
  cluster_version = "1.29"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    default = {
      desired_size = 2
      max_size     = 3
      min_size     = 1
      instance_types = ["t3.medium"]
    }
  }
}
```

### 3-4. Output (`outputs.tf`)

```hcl
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}
```

---

<!-- _class: medium -->

## 4. Terraform 실행

### 4-1. 초기화

```bash
terraform init
```

### 4-2. 실행 계획 확인

```bash
terraform plan
```

### 4-3. 배포

```bash
terraform apply -auto-approve
```

✅ 완료 후 출력 예시

```
Apply complete! Resources: 25 added, 0 changed, 0 destroyed.
Outputs:
cluster_name = "eks-demo"
cluster_endpoint = "https://XXXX.gr7.ap-northeast-2.eks.amazonaws.com"
```

---

## 5. 클러스터 연결

### 5-1. kubeconfig 업데이트

```bash
aws eks update-kubeconfig --name eks-demo --region ap-northeast-2
```

### 5-2. 노드 상태 확인

```bash
kubectl get nodes
```

✅ 출력 예시

```
NAME                                           STATUS   ROLES    AGE   VERSION
ip-10-0-1-101.ap-northeast-2.compute.internal  Ready    <none>   2m    v1.29.x
ip-10-0-2-53.ap-northeast-2.compute.internal   Ready    <none>   2m    v1.29.x
```

---

## 6. 리소스 정리

실습 종료 후 비용 방지를 위해 반드시 삭제:

```bash
terraform destroy -auto-approve
```

---

## ✅ 최종 결과

- Terraform으로 VPC + EKS 클러스터 + NodeGroup 자동 배포
    
- AWS CLI로 kubeconfig 업데이트 완료
    
- `kubectl get nodes`에서 워커 노드 상태 확인 성공
    

---

# 💻 15주차 실습 강의 – 애플리케이션 배포 자동화 (Ansible + GitHub Actions)

## 🎯 실습 목표

- Ansible로 Kubernetes Deployment + Service 배포를 자동화한다.
    
- Playbook을 실행하여 앱이 정상 동작하는지 확인한다.
    
- GitHub Actions Workflow와 연계해 CI/CD 파이프라인을 완성한다.
    

---

## 1. 환경 준비

### 1-1. Ansible 설치

`sudo apt update sudo apt install -y ansible ansible --version`

✅ `ansible [core X.Y.Z]` 출력

### 1-2. Kubernetes 모듈 설치

`ansible-galaxy collection install kubernetes.core pip install openshift`

### 1-3. 클러스터 연결 확인

`kubectl get nodes`

✅ EKS 노드 Ready 확인

---

## 2. Inventory 작성

`inventory.ini`

`[localhost] 127.0.0.1 ansible_connection=local`

👉 EKS API 서버에 로컬 kubeconfig로 접근

---

## 3. Kubernetes 매니페스트 준비

### 3-1. Deployment (`manifests/deployment.yaml`)

`apiVersion: apps/v1 kind: Deployment metadata:   name: myapp spec:   replicas: 2   selector:     matchLabels:       app: myapp   template:     metadata:       labels:         app: myapp     spec:       containers:       - name: myapp         image: nginx:1.25         ports:         - containerPort: 80`

### 3-2. Service (`manifests/service.yaml`)

`apiVersion: v1 kind: Service metadata:   name: myapp-svc spec:   selector:     app: myapp   ports:   - port: 80     targetPort: 80   type: LoadBalancer`

---

## 4. Ansible Playbook 작성

`deploy-app.yaml`

`- name: Deploy app to EKS   hosts: localhost   tasks:     - name: Apply Deployment       kubernetes.core.k8s:         state: present         src: ./manifests/deployment.yaml      - name: Apply Service       kubernetes.core.k8s:         state: present         src: ./manifests/service.yaml`

---

## 5. Playbook 실행

`ansible-playbook -i inventory.ini deploy-app.yaml`

✅ 출력 예시:

`TASK [Apply Deployment] *** changed: [127.0.0.1]  TASK [Apply Service] *** changed: [127.0.0.1]  PLAY RECAP *** 127.0.0.1 : ok=2 changed=2`

---

## 6. 배포 확인

`kubectl get deployments kubectl get pods -l app=myapp kubectl get svc myapp-svc`

✅ `EXTERNAL-IP` 확인 → 브라우저 접속 시 nginx 페이지 출력

---

## 7. GitHub Actions 연동

### 7-1. `.github/workflows/deploy.yaml`

`name: Deploy to EKS  on:   push:     branches: [ "main" ]  jobs:   deploy:     runs-on: ubuntu-latest      steps:       - name: Checkout         uses: actions/checkout@v3        - name: Configure AWS credentials         uses: aws-actions/configure-aws-credentials@v2         with:           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}           aws-region: ${{ secrets.AWS_REGION }}        - name: Update kubeconfig         run: aws eks update-kubeconfig --region ${{ secrets.AWS_REGION }} --name ${{ secrets.EKS_CLUSTER_NAME }}        - name: Install Ansible         run: |           sudo apt update && sudo apt install -y ansible           ansible-galaxy collection install kubernetes.core           pip install openshift        - name: Deploy app with Ansible         run: ansible-playbook -i inventory.ini deploy-app.yaml`

---

## 8. 정리

리소스 삭제:

`kubectl delete -f manifests/service.yaml kubectl delete -f manifests/deployment.yaml`

---

## ✅ 최종 결과

- Ansible로 Kubernetes 앱 자동 배포 성공
    
- GitHub Actions Workflow 연계 → **코드 푸시 시 자동 배포** 완성
    
- 브라우저에서 LoadBalancer `EXTERNAL-IP` 접속해 Nginx 서비스 확인
    

---

# 💻 15주차 실습 강의 – 보안 점검 및 리포트 자동화 (Trivy + Python Script)

## 🎯 실습 목표

- Trivy를 사용해 컨테이너 이미지 및 IaC 보안 점검을 수행한다.
    
- Python 스크립트(`generate_security_report.py`)로 취약점 보고서를 자동 생성한다.
    
- GitHub Actions Workflow에 보안 점검 단계를 추가해 자동화된 DevSecOps 파이프라인을 완성한다.
    

---

## 1. Trivy 설치 및 이미지 스캔

### 1-1. Trivy 설치

`sudo apt-get install wget -y wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.0_Linux-64bit.deb sudo dpkg -i trivy_0.50.0_Linux-64bit.deb`

### 1-2. nginx 이미지 스캔

`trivy image -f json -o trivy-report.json nginx:1.25`

✅ `trivy-report.json` 파일 생성

---

<!-- _class: medium -->

## 2. Python 보고서 생성 스크립트

### 2-1. `generate_security_report.py`

`import json from collections import Counter from fpdf import FPDF  # 1. JSON 로드 with open("trivy-report.json") as f:     data = json.load(f)  vulns = [] for result in data.get("Results", []):     for v in result.get("Vulnerabilities", []):         vulns.append({             "id": v.get("VulnerabilityID"),             "pkg": v.get("PkgName"),             "severity": v.get("Severity")         })  # 2. 심각도 카운트 severity_count = Counter(v["severity"] for v in vulns)  # 3. Markdown 보고서 작성 with open("security_report.md", "w") as f:     f.write("# Security Report\n\n")     f.write("## Summary\n")     for sev, count in severity_count.items():         f.write(f"- {sev}: {count}\n")     f.write("\n## Details\n")     for v in vulns:         f.write(f"- [{v['severity']}] {v['id']} ({v['pkg']})\n")  # 4. PDF 보고서 작성 pdf = FPDF() pdf.add_page() pdf.set_font("Arial", size=12) pdf.cell(200, 10, txt="Security Report", ln=True, align="C") pdf.ln(10) for sev, count in severity_count.items():     pdf.cell(200, 10, txt=f"{sev}: {count}", ln=True) pdf.ln(10) for v in vulns:     pdf.cell(200, 10, txt=f"[{v['severity']}] {v['id']} ({v['pkg']})", ln=True) pdf.output("security_report.pdf")  print("Reports generated: security_report.md, security_report.pdf")`

---

### 2-2. 의존성 설치

`pip install fpdf`

### 2-3. 스크립트 실행

`python3 generate_security_report.py`

✅ 결과 파일:

- `security_report.md`
    
- `security_report.pdf`
    

---
<!-- _class: medium -->
## 3. GitHub Actions 연동

### 3-1. `.github/workflows/security-scan.yaml`

`name: Security Scan  on:   push:     branches: [ "main" ]  jobs:   scan:     runs-on: ubuntu-latest     steps:       - name: Checkout Repository         uses: actions/checkout@v3        - name: Install Trivy         run: |           sudo apt-get install wget -y           wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.0_Linux-64bit.deb           sudo dpkg -i trivy_0.50.0_Linux-64bit.deb        - name: Run Trivy         run: trivy image -f json -o trivy-report.json nginx:1.25        - name: Install Python dependencies         run: pip install fpdf        - name: Generate Security Report         run: python3 generate_security_report.py        - name: Upload Report Artifacts         uses: actions/upload-artifact@v3         with:           name: security-reports           path: |             security_report.md             security_report.pdf`

---

## 4. Workflow 실행 및 확인

1. `git push origin main`
    
2. GitHub → **Actions 탭**에서 `Security Scan` 실행 확인
    
3. 실행 완료 후 **Artifacts → security-reports.zip** 다운로드
    
4. PDF와 Markdown 보고서 확인
    

---

## 5. 정리

- Trivy로 이미지 취약점 점검
    
- Python 스크립트로 보고서 자동 생성
    
- GitHub Actions로 DevSecOps 파이프라인 완성
    

---

## ✅ 최종 결과

- **Terraform → 인프라 자동화(EKS)**
    
- **Ansible → 앱 배포 자동화**
    
- **GitHub Actions → CI/CD**
    
- **Trivy + Python → 보안 점검 & 자동 보고**
    



---


# 📦 15주차 Mini Project – Terraform + Ansible + CI/CD + 보안 점검 파이프라인 완성

## 🎯 최종 목표

- Terraform으로 AWS EKS 클러스터 자동 배포
    
- Ansible + GitHub Actions로 앱 배포 자동화
    
- Trivy + Python으로 보안 점검 및 보고서 자동화
    
- DevSecOps 기반 **쿠버네티스 실전 배포 파이프라인** 완성
    

---

<!-- _class: small -->
## 1. 인프라 자동화 (Terraform → EKS 배포)

### 1-1. 프로젝트 디렉토리 생성

```bash
mkdir ~/terraform-eks-project
cd ~/terraform-eks-project
```
---
<!-- _class: small -->
### 1-2. Terraform 코드 작성 (`provider.tf`, `vpc.tf`, `eks.tf`, `outputs.tf`)

```hcl
provider "aws" {
  region = "ap-northeast-2"
}

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

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.3"

  cluster_name    = "eks-demo"
  cluster_version = "1.29"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    default = {
      desired_size = 2
      max_size     = 3
      min_size     = 1
      instance_types = ["t3.medium"]
    }
  }
}

output "cluster_name" {
  value = module.eks.cluster_name
}
```

---
### 1-3. 실행

```bash
terraform init
terraform apply -auto-approve
aws eks update-kubeconfig --name eks-demo --region ap-northeast-2
kubectl get nodes
```

✅ EKS 노드 Ready 확인

---

<!-- _class: medium -->
## 2. 애플리케이션 배포 자동화 (Ansible + GitHub Actions)

### 2-1. 매니페스트 작성

`manifests/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.25
        ports:
        - containerPort: 80
```
---
`manifests/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

---
### 2-2. Ansible Playbook 작성 (`deploy-app.yaml`)

```yaml
- name: Deploy app to EKS
  hosts: localhost
  tasks:
    - name: Apply Deployment
      kubernetes.core.k8s:
        state: present
        src: ./manifests/deployment.yaml
    - name: Apply Service
      kubernetes.core.k8s:
        state: present
        src: ./manifests/service.yaml
```

실행:

```bash
ansible-playbook -i inventory.ini deploy-app.yaml
kubectl get svc myapp-svc
```

✅ `EXTERNAL-IP` 접속 → nginx 페이지 확인

---
<!-- _class: medium -->
### 2-3. GitHub Actions CI/CD Workflow (`.github/workflows/deploy.yaml`)

```yaml
name: Deploy to EKS

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Update kubeconfig
        run: aws eks update-kubeconfig --region ${{ secrets.AWS_REGION }} --name ${{ secrets.EKS_CLUSTER_NAME }}

      - name: Install Ansible
        run: |
          sudo apt update && sudo apt install -y ansible
          ansible-galaxy collection install kubernetes.core
          pip install openshift

      - name: Deploy with Ansible
        run: ansible-playbook -i inventory.ini deploy-app.yaml
```

---

<!-- _class: medium -->
## 3. 보안 점검 및 리포트 자동화 (Trivy + Python)

### 3-1. 이미지 스캔

```bash
trivy image -f json -o trivy-report.json nginx:1.25
```

---
<!-- _class: small -->
### 3-2. Python 스크립트 (`generate_security_report.py`)

```python
import json
from collections import Counter
from fpdf import FPDF

with open("trivy-report.json") as f:
    data = json.load(f)

vulns = []
for result in data.get("Results", []):
    for v in result.get("Vulnerabilities", []):
        vulns.append({"id": v["VulnerabilityID"], "pkg": v["PkgName"], "severity": v["Severity"]})

severity_count = Counter(v["severity"] for v in vulns)

with open("security_report.md", "w") as f:
    f.write("# Security Report\n\n## Summary\n")
    for sev, count in severity_count.items():
        f.write(f"- {sev}: {count}\n")
    f.write("\n## Details\n")
    for v in vulns:
        f.write(f"- [{v['severity']}] {v['id']} ({v['pkg']})\n")

pdf = FPDF(); pdf.add_page(); pdf.set_font("Arial", size=12)
pdf.cell(200, 10, "Security Report", ln=True, align="C"); pdf.ln(10)
for sev, count in severity_count.items(): pdf.cell(200, 10, f"{sev}: {count}", ln=True)
pdf.ln(10)
for v in vulns: pdf.cell(200, 10, f"[{v['severity']}] {v['id']} ({v['pkg']})", ln=True)
pdf.output("security_report.pdf")
```

---
실행:

```bash
pip install fpdf
python3 generate_security_report.py
```

✅ `security_report.md`, `security_report.pdf` 생성


---
<!-- _class: small -->
### 3-3. GitHub Actions Workflow (`.github/workflows/security-scan.yaml`)

```yaml
name: Security Scan

on: [push]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Trivy
        run: |
          sudo apt-get install wget -y
          wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.0_Linux-64bit.deb
          sudo dpkg -i trivy_0.50.0_Linux-64bit.deb

      - name: Run Trivy
        run: trivy image -f json -o trivy-report.json nginx:1.25

      - name: Install Python
        run: pip install fpdf

      - name: Generate Report
        run: python3 generate_security_report.py

      - name: Upload Reports
        uses: actions/upload-artifact@v3
        with:
          name: security-reports
          path: |
            security_report.md
            security_report.pdf
```

---

## ✅ 최종 결과

- **Terraform** → AWS EKS 자동 배포
    
- **Ansible** → Kubernetes 앱 자동 배포
    
- **GitHub Actions** → CI/CD 자동화
    
- **Trivy + Python** → 보안 점검 & 보고서 자동 생성
    
