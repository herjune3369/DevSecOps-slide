

# 📝 7주차 실습 가이드 – Ansible 기초

(Inventory, Playbook, Role 구조)

---

## 🎯 이번 주 학습 목표

- Ansible이 왜 필요한지 이해한다.
    
- **Inventory** (내가 관리할 서버 주소록) 개념을 익힌다.
    
- **Playbook** (자동 실행할 작업 지시서)을 작성한다.
    
- **Role** (Playbook을 폴더 구조로 정리한 형태)을 맛본다.
    
- 직접 EC2에 Flask를 설치해본다.
    

---

## 1️⃣ 왜 Ansible을 쓰는가?

- 지금까지는 Terraform으로 **서버(EC2)까지 자동으로 생성**했습니다.
    
- 그런데 서버 안에 들어가서:
    
    - Python 설치
        
    - Flask 설치
        
    - 앱 실행  
        … 이런 건 매번 사람이 `ssh`로 들어가서 명령어를 쳐야 했습니다.
        

👉 Ansible은 이 부분을 자동으로 해주는 **서버 집사** 같은 도구입니다.

---

## 2️⃣ Ansible 기본 개념

1. **Inventory**
    
    - Ansible이 관리할 서버 목록.
        
    - 서버 주소록이라고 생각하면 됨.
        
2. **Playbook**
    
    - 서버에서 실행할 작업들을 적어놓은 지시문.
        
    - 예: “이 서버에 Python 설치 → Flask 설치 → 앱 실행해라”
        
3. **Role**
    
    - Playbook을 체계적으로 정리해놓은 폴더 구조.
        
    - 프로젝트가 커질 때 꼭 필요.
        

---

## 3️⃣ 사전 준비

1. **로컬에 Ansible 설치**  
    (Ubuntu, Mac 공통)
    
    ```bash
    sudo apt-get update && sudo apt-get install -y ansible
    ansible --version
    ```
    
2. **SSH 키 준비**
    
    - 지난주 Terraform이 만든 키 `~/.ssh/terraform-key.pem` 사용
        
    - 권한 확인:
        
        ```bash
        chmod 600 ~/.ssh/terraform-key.pem
        ```
        
3. **EC2 Public IP 확인**
    
    - AWS 콘솔 → EC2 → Public IP 확인
        
    - 예시: `43.201.0.251`
        

---

## 4️⃣ Inventory 작성하기

📄 **inventory.ini**

```ini
[webserver]
43.201.0.251 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

- `webserver` = 그룹 이름
    
- `43.201.0.251` = EC2 퍼블릭 IP
    
- `ansible_user=ubuntu` = 우분투 서버 기본 계정
    
- `ansible_ssh_private_key_file` = 접속할 키 파일
    

👉 이렇게 적어두면, Ansible은 이제 `webserver`라는 이름만 보고도 서버에 접속할 수 있습니다.

---

## 5️⃣ 첫 번째 Playbook

📄 **playbook.yml**

```yaml
- name: Setup Flask Web Server
  hosts: webserver
  become: true   # root 권한 사용
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
👉 의미:

- webserver 그룹에 있는 서버에 들어가서,
    
    1. apt 업데이트
        
    2. python3 + pip 설치
        
    3. Flask 설치
        

---

## 6️⃣ 실행해보기
---

## 1. 인벤토리 확인

```bash
ansible-inventory -i inventory.ini --list
```

- **inventory.ini** = Ansible이 관리할 서버 주소록 파일
    
- 이 명령은 “내가 관리할 서버 주소록이 제대로 읽히는지” 확인하는 겁니다.
    

👉 비유: **전화번호부를 열어서 ‘번호가 제대로 저장돼 있나?’ 확인하는 단계**

예:

- `webserver` 그룹 안에 `43.201.0.251`이 보이면 → “아, 서버 목록을 Ansible이 제대로 인식했구나”
    

---

## 2. 서버 핑 확인

```bash
ansible -i inventory.ini all -m ping
```

- `-m ping`은 “서버에 진짜 접속이 되는지 테스트해줘”라는 뜻
    
- 네트워크 핑이 아니라, **Ansible이 SSH로 로그인해서 응답 받는 테스트**임
    

👉 비유: **전화번호부에 있는 번호로 전화를 걸어 “여보세요, 들리세요?” 물어보는 단계**

예:

- 응답이 `pong` 나오면 → “연결 성공, 접속 가능”
    
- 에러가 나오면 → 키 파일 경로, 사용자 계정, 보안그룹 설정 문제
    

---

## 3. Playbook 실행

```bash
ansible-playbook -i inventory.ini playbook.yml
```

- **playbook.yml** = 서버에서 할 일을 적어둔 “자동화 시나리오”
    
- 이 명령을 실행하면, Ansible이 서버에 접속해서 거기에 적힌 작업을 차례대로 수행합니다.
    

👉 비유: **전화를 건 뒤 “밥 해줘 → 청소해줘 → 불 끄고 나와”라고 지시하면, 집사가 그대로 해주는 단계**

예:

- apt 업데이트 → Python 설치 → Flask 설치
    
- 실행이 끝나면, EC2 안에는 Flask 환경이 갖춰짐
    

---

## ✅ 정리

1. **인벤토리 확인** = 주소록에 서버가 제대로 적혔는지 확인
    
2. **서버 핑 확인** = 서버에 진짜 접속이 되는지 시험
    
3. **Playbook 실행** = 접속된 서버에서 지시사항(설치, 설정)을 자동으로 실행
    


---

## 7️⃣ Role 맛보기

- Playbook이 많아지면 지저분해지니까, **Role 구조**로 정리합니다.
    

📂 **폴더 구조**

```
roles/
 └─ web/
     ├─ tasks/main.yml
     ├─ handlers/main.yml
     ├─ templates/
     └─ files/
```
---
📄 **roles/web/tasks/main.yml**

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

📄 **playbook.yml (Role 버전)**

```yaml
- name: Configure Flask Webserver with Role
  hosts: webserver
  become: true
  roles:
    - web
```

---

## 8️⃣ 학습 정리

- Terraform은 “서버를 만든다.”
    
- Ansible은 “서버 안에 프로그램을 깔고, 설정하고, 돌린다.”
    
- **Inventory** = 주소록
    
- **Playbook** = 지시서
    
- **Role** = 깔끔한 구조화
    

---

## ✅ 과제

1. Inventory에 EC2를 **두 대** 넣고 동시에 Flask 설치하기
    
2. Role에 `templates/app.py.j2` 추가해서 Flask 앱 자동 배포하기
    
3. Handlers를 이용해 `systemctl restart flask` 서비스 재시작 자동화 시도
    


