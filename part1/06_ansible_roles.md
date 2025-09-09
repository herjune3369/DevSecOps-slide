📘 Part 2. Ansible 자동화 & CI/CD

---

## 📌 Part 2. 학습 주차 개요

- **6-7주차**: Ansible 기초 (Inventory, Playbook, Role 구조)
    
- **8-9주차**: Flask + DB 자동 배포 (Ansible)
    
- **10주차**: GitHub Actions 기본 구조 (Workflow, Event, Job, Step)
    
- **11주차**: GitHub Actions + Terraform + Ansible → CI/CD 자동화
- 
- 12주차: GitHub Actions + Terraform + output → ansibel 자동전달
    
- **13~14주차**: 🚀 **Mini Project ③ – AI 사주 앱 CI/CD 완성**
    

---

## 🎯 학습 목표

- **Ansible**을 활용한 서버 구성 자동화 이해
    
- Flask 웹앱 + RDS DB를 **자동 배포**
    
- GitHub Actions로 **CI/CD 파이프라인 구축**
    
- Terraform, Ansible, GitHub Actions의 **연계 자동화 실습**
    
- 최종적으로 **AI 사주 앱 CI/CD 환경 완성**
    

---

## 🌐 왜 Ansible & GitHub Actions인가?

- **Ansible**: Agentless, YAML 기반 → 쉽고 직관적인 서버 자동화
    
- **GitHub Actions**: GitHub과 완벽 연계 → CI/CD 자동화 표준
    
- Terraform + Ansible + GitHub Actions → **완전 자동화 DevOps 파이프라인**
    


---

#  6주차 실습 가이드 – Ansible 기초 (Inventory, Playbook, Role)

## 🎯 실습 목표

- Ansible의 기본 구조와 동작 원리를 이해한다.
    
- Inventory(관리 대상 서버 목록) 작성법을 익힌다.
    
- Playbook을 작성해 원격 EC2에 패키지 설치 자동화한다.
    
- Role 구조를 활용해 **재사용 가능한 자동화 코드**를 만든다.
    

---

## 📌 1. 사전 준비

1. **Ansible 설치 (로컬 PC 또는 관리 서버)**
    
    ```bash
    sudo apt-get update && sudo apt-get install -y ansible
    ansible --version
    ```
    
2. **SSH Key 확인**
    
    - 지난 10주차에서 Terraform이 생성한 `~/.ssh/terraform-key.pem` 사용
        
    - 퍼미션 확인:
        
        ```bash
        chmod 600 ~/.ssh/terraform-key.pem
        ```

`~/.ssh/terraform-key.pem`으로 옮겨두면 Ansible이랑 SSH 접속이 제일 깔끔해집니다.

---

## 🛠 실행 절차

1. 키 파일 위치 확인 (예: `~/politec-book/terraform-key.pem`)
    
    ```bash
    ls -l ~/politec-book/terraform-key.pem
    ```
    
2. `~/.ssh` 폴더가 없으면 생성
    
    ```bash
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    ```
    
3. 키 이동 & 권한 설정
    
    ```bash
    mv ~/politec-book/terraform-key.pem ~/.ssh/terraform-key.pem
    chmod 600 ~/.ssh/terraform-key.pem
    ```
    
4. 잘 옮겨졌는지 확인
    
    ```bash
    ls -l ~/.ssh/terraform-key.pem
    ```

---

## ✅ 정리

- 이제 **SSH 접속**은:
    
    ```bash
    ssh -i ~/.ssh/terraform-key.pem ubuntu@<EC2_PUBLIC_IP>
    ```
    
- **Ansible inventory.ini**도 그대로 사용 가능:
    
    ```ini
    [webserver]
    43.201.0.251 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
    ```
    


---

## 📌 2. 디렉토리 구조

```bash
ansible/
 ├─ inventory.ini       # 관리 대상 서버 목록
 ├─ playbook.yml        # 실행할 작업 정의
 └─ roles/
     └─ web/
         ├─ tasks/main.yml
         ├─ handlers/main.yml
         ├─ templates/
         └─ files/
```

---

## 📌 3. Inventory 작성

**inventory.ini**

```ini
[webserver]
43.201.0.251 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

- `webserver` → 그룹 이름
    
- `43.201.0.251` → EC2 퍼블릭 IP
    
- `ansible_user=ubuntu` → Ubuntu 기본 계정
    
- `ansible_ssh_private_key_file` → SSH 접속 키
    

---

## 📌 4. 첫 번째 Playbook

**playbook.yml**

```yaml
- name: Setup Flask Web Server
  hosts: webserver
  become: true
  tasks:
    - name: Update apt
      apt:
        update_cache: yes

    - name: Install Python3 and pip
      apt:
        name:
          - python3
          - python3-pip
        state: present

    - name: Install Flask
      pip:
        name: flask
```

---

## 📌 5. Role 구조 예시

**roles/web/tasks/main.yml**

```yaml
- name: Install required packages
  apt:
    name:
      - python3
      - python3-pip
    state: present
    update_cache: yes

- name: Install Flask
  pip:
    name: flask
```
---
**roles/web/handlers/main.yml**

```yaml
- name: restart flask
  service:
    name: flask
    state: restarted
```

**playbook.yml (Role 활용)**

```yaml
- name: Configure Flask Webserver with Role
  hosts: webserver
  become: true
  roles:
    - web
```

---

## 📌 6. 실행

```bash
ansible-inventory -i inventory.ini --list
ansible -i inventory.ini all -m ping

ansible-playbook -i inventory.ini playbook.yml
```

- 첫 실행에서 SSH known_hosts 경고 나오면 `yes` 입력
    
- Flask 설치 완료 후, EC2에 접속해 `python3 -m flask --version` 확인
    

---

## 📌 7. 학습 정리

- **Inventory**: 어떤 서버를 관리할지 정의 (IP, 접속 계정, 키 파일)
    
- **Playbook**: 서버에서 실행할 작업(Task) 목록
    
- **Role**: Playbook을 모듈화/재사용 가능하게 만든 구조
    
- **결과물**: Ansible로 원격 EC2에 Flask 자동 설치 성공
    

---

## ✅ 과제

1. Inventory에 EC2를 2대 추가해 동시에 Flask 설치하기
    
2. Role에 `templates/app.py.j2` 추가해 Flask 앱 배포까지 자동화
    
3. Handlers를 활용해 `systemctl restart flask` 같은 서비스 재시작 자동화 시도
    


