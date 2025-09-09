# 📝 8주차 실습 가이드 – Ansible로 Flask + DB 자동 배포

---

## 🎯 학습 목표

- Ansible로 EC2에 Flask 앱을 자동으로 배포한다.
    
- RDS(MySQL)와 연결할 수 있는 환경을 만든다.
    
- Playbook/Role 구조에서 **templates**와 **handlers**까지 활용해 본다.
    
- 최종적으로 ALB DNS로 접속했을 때 **AI 사주 앱** 실행 결과를 확인한다.
    

---

## 1️⃣ Ansible 확장 개념

1. **템플릿(templates)**
    
    - `Jinja2` 문법(`{{ 변수 }}`)을 이용해 동적으로 파일 생성 가능.
        
    - 예: DB 접속정보를 `app.py` 안에 자동으로 채워넣음.
        
2. **핸들러(handlers)**
    
    - 특정 작업 후에만 실행되는 “후속 동작” 정의.
        
    - 예: `app.py`가 새로 배포되면 Flask 서비스를 재시작.
        

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
         ├─ templates/app.py.j2
         └─ files/
```

---

## 3️⃣ Inventory 확인

📄 **inventory.ini**

```ini
[webserver]
13.209.9.117 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/terraform-key.pem
```

---

## 4️⃣ Flask 앱 템플릿 작성

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

---
<!-- _class: medium -->

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
  notify: restart flask
```

---

## 6️⃣ Handlers 작성

📄 **roles/web/handlers/main.yml**

```yaml
- name: restart flask
  shell: |
    pkill -f app.py || true
    nohup python3 /home/ubuntu/app.py &
```

---

## 7️⃣ Playbook 실행 파일

📄 **playbook.yml**

```yaml
- name: Deploy Flask Webserver with DB Connection
  hosts: webserver
  become: true
  vars:
    rds_endpoint: "terraform-rds-endpoint.rds.amazonaws.com"  # Terraform outputs에서 가져옴
    db_user: "admin"
    db_password: "Passw0rd123!"
    db_name: "sajuapp"
  roles:
    - web
```

---

## 8️⃣ 실행 방법

```bash
ansible-playbook -i inventory.ini playbook.yml
```

- 설치 및 배포 완료 후:
    
    - ALB DNS로 접속:
        
        ```
        http://<ALB_DNS>
        ```
        
        → `"AI 사주 앱 – Ansible 자동 배포 성공!"`
        
    - DB 테스트:
        
        ```
        http://<ALB_DNS>/db
        ```
        
        → `"DB 연결 성공! 현재 시간: 2025-09-02 12:34:56"` (예시)
        

---

## 9️⃣ 학습 정리

- **Terraform** = 서버와 DB를 만든다.
    
- **Ansible** = 서버 안에 Flask 앱을 배포하고, DB 연결 설정까지 자동으로 해준다.
    
- **템플릿** = 앱 파일을 동적으로 생성.
    
- **핸들러** = 앱이 바뀔 때만 자동 재시작.
    
- 결과: 클릭 한 번에 “웹서버 + DB + 앱 배포” 완성!
    

---

## ✅ 과제

1. Flask 앱에 `/fortune` 라우트 추가 → “오늘의 운세: 행운의 하루!” 출력
    
2. DB 연결 실패 시, 로그를 `/home/ubuntu/app-error.log`에 저장하도록 수정
    
3. ALB 헬스체크 경로를 `/db`로 바꿔서 DB 연결 여부도 확인
    

