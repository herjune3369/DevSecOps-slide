
# 📝 9주차 실습 가이드 – Ansible로 Flask + DB 자동 배포

---

## 🎯 학습 목표

- Ansible로 서버에 Flask 앱을 자동 배포한다.
    
- RDS(MySQL)에 연결해서 단순한 쿼리 테스트까지 해본다.
    
- **템플릿(templates)**과 **핸들러(handlers)** 개념을 실제로 사용해본다.
    
- 최종적으로 ALB DNS 접속 시 **Flask 앱 실행 결과**를 확인한다.
    

---

## 1️⃣ Ansible에서 이번에 배우는 핵심

1. **템플릿(template)**
    
    - Ansible은 `Jinja2` 문법을 이용해서 변수 값을 코드 안에 끼워넣을 수 있습니다.
        
    - 즉, DB endpoint 같은 정보를 하드코딩하지 않고, 변수를 넘겨서 자동으로 채워넣습니다.
        
2. **핸들러(handler)**
    
    - 특정 작업(Task)이 실행되면 자동으로 불리는 후속 작업입니다.
        
    - 예: `app.py`가 새로 배포되면 Flask 앱을 재시작한다.
        

👉 학생들은 이번 주차에서 “Ansible이 파일도 배포하고, 서비스까지 관리한다”는 걸 경험합니다.

---

## 2️⃣ 디렉토리 구조

```
ansible/
 ├─ inventory.ini
 ├─ playbook.yml
 └─ roles/
     └─ web/
         ├─ tasks/main.yml
         ├─ handlers/main.yml
         └─ templates/app.py.j2
```

---

## 3️⃣ Inventory (서버 주소록)

📄 **inventory.ini**

```ini
[webserver]
13.209.9.117 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

- `webserver` → 그룹 이름
    
- `13.209.9.117` → EC2 퍼블릭 IP
    
- `ansible_user=ubuntu` → 우분투 서버 로그인 계정
    
- `ansible_ssh_private_key_file` → 접속 키 파일 경로
    

👉 여기서 학생들은 “Ansible은 IP 대신 그룹(webserver) 이름으로 서버를 관리한다”는 걸 배우게 됩니다.

---

## 4️⃣ Flask 앱 템플릿

📄 **roles/web/templates/app.py.j2**

```python
from flask import Flask
import pymysql

app = Flask(__name__)

@app.route("/")
def hello():
    return "AI 사주 앱 – Ansible 자동 배포 성공!"
```
---
```python
@app.route("/db")
def test_db():
    try:
        conn = pymysql.connect(
            host="{{ rds_endpoint }}",
            user="{{ db_user }}",
            password="{{ db_password }}",
            database="{{ db_name }}"
        )
        cursor = conn.cursor()
        cursor.execute("SELECT NOW()")
        result = cursor.fetchone()
        conn.close()
        return f"DB 연결 성공! 현재 시간: {result}"
    except Exception as e:
        return f"DB 연결 실패: {str(e)}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

- `{{ rds_endpoint }}` 같은 값은 Ansible이 **변수로 전달**합니다.
- 따라서 코드 안에 민감한 정보를 직접 쓰지 않아도 됩니다.
    

---

## 5️⃣ Tasks 작성

📄 **roles/web/tasks/main.yml**

```yaml
- name: Install required packages
  apt:
    name:
      - python3
      - python3-pip
      - python3-pymysql
    state: present
    update_cache: yes

- name: Install Flask and PyMySQL
  pip:
    name:
      - flask
      - pymysql

- name: Deploy Flask app from template
  template:
    src: app.py.j2
    dest: /home/ubuntu/app.py
    mode: '0644'
    
  notify: restart flask  --> 에러
```

- “서버에 직접 접속해서 설치하지 않아도, Ansible이 알아서 Python과 Flask를 설치
    
- 마지막 `notify`는 **핸들러 호출** → 파일이 새로 배포되면 Flask 앱을 재시작합니다.
    

---

## 6️⃣ Handlers 작성

📄 **roles/web/handlers/main.yml**

```yaml
- name: restart flask
  shell: |
    pkill -f app.py || true
    nohup python3 /home/ubuntu/app.py &   -->  이줄 에러
```

👉 의미:

- 기존에 돌던 Flask 프로세스가 있으면 종료(`pkill -f app.py`)
    
- 새 Flask 앱을 백그라운드에서 실행(`nohup ... &`)
    


---

## 7️⃣ Playbook 작성

📄 **playbook.yml**

```yaml
- name: Deploy Flask Webserver with DB Connection
  hosts: webserver
  become: true
  vars:
    rds_endpoint: "terraform-rds-endpoint.rds.amazonaws.com"  # Terraform outputs에서 가져오기
    db_user: "admin"
    db_password: "Passw0rd123!"
    db_name: "sajuapp"
  roles:
    - web
```

---

## 8️⃣ 실행 절차

1. 실행
    
    ```bash
    ansible-playbook -i inventory.ini playbook.yml
    ```
    
2. 결과 확인
    
    - ALB DNS로 접속:
        
        ```
        http://<ALB_DNS>
        ```
        
        → `"AI 사주 앱 – Ansible 자동 배포 성공!"`
        
    - DB 테스트:
        
        ```
        http://<ALB_DNS>/db
        ```
        
        → `"DB 연결 성공! 현재 시간: 2025-09-02 12:34:56"`
        

---

## 9️⃣ 학습 정리

- **Terraform** = 서버와 DB를 만든다.
    
- **Ansible** = 서버 안에 Flask 앱을 설치·배포·실행까지 해준다.
    
- **템플릿** = 민감정보(DB endpoint 등)를 코드에 직접 쓰지 않고 변수로 관리한다.
    
- **핸들러** = 앱이 수정되면 자동 재시작까지 해준다.
    
- 이제 “코드 한 줄로 인프라 + 앱 + DB 전체 배포” 경험을 하게 된다.
    

---

## ✅ 과제

1. `/db` 라우트를 `/health`로 바꾸고 ALB Health Check 경로를 `/health`로 설정
    
2. DB 쿼리문을 `SELECT NOW()` 대신 `SELECT '운세 테스트'`로 바꿔보기
    
3. 앱 실행 로그를 `/home/ubuntu/flask.log`로 남기도록 handlers 수정
    


