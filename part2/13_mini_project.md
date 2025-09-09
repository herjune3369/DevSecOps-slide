
# 13주차: 프로젝트 구현 & 테스트 — “한 번에 따라하기”(실습, 3시간)

---

## 🎯 목표

- 업로드된 **AI 사주 앱**을 기준으로 인프라 배포 → 앱 배포 → 보안 스캔까지 한 번에.
    
- 결과물: 실행 중인 앱 URL, Trivy JSON 리포트 3종, 팀 리드미/스크린샷.
    

---

## 0) 사전 준비 (5분)

- 팀 GitHub Repo 준비(또는 기존 Repo 사용): `app/`, `terraform/`, `ansible/`, `.github/workflows/`, `reports/` 구조.
    
- 로컬(또는 관리 VM)에 다음 설치: **AWS CLI, Terraform(≥1.4), Ansible(≥2.15), Python3, Trivy, Docker**.
    

> 오늘 모든 산출물은 `reports/`에 저장합니다.

---


## 1) 환경 세팅 (25분)

### 1-1. 코드 배치 & 구조 확인

```bash
git clone https://github.com/<team-org>/<team-project>.git
cd <team-project>
tree -L 2
```

예상:

```
app/                 # Flask AI 사주 앱
terraform/           # IaC
ansible/             # 배포 자동화
.github/workflows/   # CI/CD
reports/             # 리포트 폴더(직접 생성)
```
---
### 1-2. AWS 인증

```bash
aws configure  # region: ap-northeast-2, output: json
aws sts get-caller-identity
```

### 1-3. 도구 버전 확인

```bash
terraform -version
ansible --version
trivy --version
docker --version
```

### 1-4. (선택) 앱 로컬 기동 점검

```bash
cd app
pip install -r requirements.txt
python app.py
# 브라우저: http://127.0.0.1:5000
```

---
<!-- _class: medium -->

## 2) Terraform 배포 (40분)

### 2-1. 초기화 & 플랜

```bash
cd terraform
terraform init
terraform plan
```

### 2-2. 적용

```bash
terraform apply -auto-approve
```

### 2-3. 출력값 확인 → 기록

```bash
terraform output
# 반드시 메모: ec2_public_ip, (있다면) rds_endpoint, vpc_id 등
```

> **문제해결 미션 A**
> 
> - `apply` 실패 시: 자격증명/리전 확인 → `aws sts get-caller-identity` 재확인
>     
> - VPC/서브넷 관련 에러: 기본 VPC/서브넷 의존 리소스 여부 확인, 변수 `subnet_id`/`vpc_id` 설정.
>     

---


## 3) Ansible로 앱 배포 (40분)

### 3-1. 인벤토리 작성

`ansible/inventories/hosts.ini`

```
[app]
<ec2_public_ip>
```

### 3-2. SSH 접속 확인

```bash
ssh -i <your_key.pem> ubuntu@<ec2_public_ip>
# 접속 OK 확인 후 종료
```
---
### 3-3. (필요 시) Ansible 변수 설정

`ansible/group_vars/app.yml` (예시)

```yaml
app_dir: /opt/ai-saju
repo_dir: /home/ubuntu/<team-project>
flask_port: 5000
db_host: "{{ lookup('env','DB_HOST') | default('localhost') }}"
```

### 3-4. 플레이북 실행

```bash
cd ansible
ansible-playbook -i inventories/hosts.ini playbook.yml
```
---
### 3-5. 서비스 확인

```bash
curl -I http://<ec2_public_ip>:5000
# 200 OK 확인
```

브라우저 접속: `http://<ec2_public_ip>:5000`

> **문제해결 미션 B**
> 
> - 방화벽/SG: 인바운드에 TCP 5000(또는 Nginx 사용 시 80) 허용 확인.
>     
> - 권한 문제: `become: true` 설정, 파일/디렉터리 권한 확인.
>     
> - 파이썬 의존성: `pip install -r requirements.txt` 로그 재검토.
>     

---

## 4) Trivy 보안 점검 (35분)

### 4-1. 컨테이너 이미지 스캔(CWPP)

```bash
mkdir -p ../reports
trivy image -f json -o ../reports/python.json python:3.9
```

### 4-2. IaC 스캔(CSPM – Terraform)

```bash
cd ../..
trivy config -f json -o reports/terraform.json ./terraform
```
---
### 4-3. AWS 계정 스캔(CSPM – Account)

```bash
trivy aws --region ap-northeast-2 -f json -o reports/aws.json
```

> **문제해결 미션 C**
> 
> - 결과가 비어보이면 `trivy image --refresh` 또는 `trivy --cache-dir` 초기화 후 재시도.
>     
> - AWS 스캔 권한 부족 시: 해당 IAM 권한(읽기 권한) 추가 후 재실행.
>     

---

<!-- _class: medium -->

## 5) 중간 점검 & 팀 리포트 초안 (20분)

### 5-1. Top 취약점 뽑기

```bash
# 예: Critical 1건 확인용(간단 도식)
cat reports/python.json | jq '.Results[].Vulnerabilities[] | select(.Severity=="CRITICAL") | .VulnerabilityID' | head
```

### 5-2. 팀 리드미(초안)

`reports/README-13w.md` 초안 템플릿:

```markdown
# 13주차 결과 요약

## 인프라
- EC2: <ec2_public_ip>
- RDS: <rds_endpoint>(있다면)

## 앱
- URL: http://<ec2_public_ip>:5000
- 배포 방법: Ansible playbook.yml

## 보안 스캔 결과(요약)
- Image: Critical X, High Y
- Terraform: HIGH Z개 (예: SG 0.0.0.0/0)
- AWS: HIGH/CRITICAL N개 (예: Root MFA 미사용)

## 개선 후보(초안)
- SG 제한, RDS 암호화, `.trivyignore` 후보(다음 주 다룸)
```

---

## 6) 확장 미션 (선택, 20분)

- **A. 인스턴스 타입 변경**: `terraform/variables.tf`에서 `t3.micro` → `t3.small`, `apply`로 반영.
    
- **B. 시스템 서비스화**: `systemd`로 Flask를 서비스 등록(재부팅 후 자동 시작 확인).
    
- **C. Nginx 앞단 배치**: 80 포트로 리버스 프록시 구성.
    
- **D. 추가 이미지 스캔**: `nginx:alpine`, `redis:latest` 스캔 후 비교표 추가.
    

---

## 7) 마무리 체크리스트 (5분)

-  **EC2 IP**로 앱 접속 성공 (또는 Nginx/80 경유 접속)
    
-  `reports/python.json`, `reports/terraform.json`, `reports/aws.json` 생성
    
-  `reports/README-13w.md`에 요약/스크린샷 첨부
    
-  문제/개선사항 ToDo 정리(14주차 문서화에 사용)
    

---

## 제출물

1. 접속 URL과 스크린샷(앱 메인 화면)
    
2. `reports/` 폴더(스캔 JSON 3종 + README-13w.md)
    
3. 오류가 있었다면 **원인/해결 로그**(간단 메모 가능)
    



---
