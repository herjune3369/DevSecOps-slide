
# 📝 10주차 실습 가이드 – GitHub Actions 기본 구조 (Workflow, Event, Job, Step)

---

## 🎯 학습 목표

- GitHub Actions의 동작 원리를 이해한다.
    
- Workflow / Event / Job / Step의 관계를 배운다.
    
- 간단한 CI 파이프라인(`hello-world`)을 만들어본다.
    
- push할 때마다 자동으로 테스트/빌드가 실행되는 경험을 한다.
    

---

## 1️⃣ GitHub Actions란?

- **CI/CD(지속적 통합/지속적 배포)**를 GitHub 자체에서 지원하는 기능
    
- 코드를 push 하면 → GitHub Actions가 자동으로 서버를 띄워 명령어 실행
    
- Terraform/Ansible과 연결하면 나중에는 **코드 → 자동 인프라 배포**까지 가능
    

👉  “GitHub에 코드 올리면 자동으로 뭔가 돌아간다”는 **마법 같은 경험**

---

## 2️⃣ GitHub Actions 기본 개념

1. **Workflow**
    
    - `.github/workflows/` 안에 있는 `.yml` 파일
        
    - “자동화 시나리오”
        
2. **Event**
    
    - Workflow를 언제 실행할지 결정
        
    - 예: `push`, `pull_request`, `schedule`

---
3. **Job**
    
    - 실제 실행 단위
        
    - 한 Workflow 안에 여러 Job 가능 (병렬 실행 가능)
        
4. **Step**
    
    - Job 안에서 실행되는 세부 명령어
        
    - 예: `checkout`, `run: echo "Hello"`
        

---

## 3️⃣ 디렉토리 구조

```
my-first-actions/
 ├─ app.py
 └─ .github/
     └─ workflows/
         └─ hello.yml
```

---

## 4️⃣ Hello World Workflow 만들기

📄 **.github/workflows/hello.yml**

```yaml
name: Hello GitHub Actions

on: 
  push:            # push 이벤트 발생 시 실행
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest   # 실행 환경

    steps:
      - name: Checkout code
        uses: actions/checkout@v3   # 깃허브 코드 가져오기

      - name: Run Hello World
        run: echo "🎉 Hello GitHub Actions! CI/CD 시작!"
```

---

## 5️⃣ 실행 방법

1. 로컬에서 GitHub repo 생성 후 푸시
    
    ```bash
    git init
    git add .
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/<username>/my-first-actions.git
    git push -u origin main
    ```
    
2. GitHub repo → Actions 탭 확인
    
    - “Hello GitHub Actions” 워크플로우가 실행되는지 확인
        
3. 로그 확인
    
    - `🎉 Hello GitHub Actions! CI/CD 시작!` 출력 확인
        

---

## 6️⃣ 학습 정리

- **Workflow** = 자동화 시나리오
    
- **Event** = 언제 실행할지
    
- **Job** = 실행 단위
    
- **Step** = 실제 명령어
    

👉 “내가 코드를 올리면 자동으로 무언가 실행되는” CI/CD의 기본 감각을 익힌다.

---

## ✅ 과제

1. `app.py`를 만들어서 **“Hello Flask!” 출력**하는 간단한 앱 작성
    
2. GitHub Actions Workflow에 Python 환경 추가:
    
    ```yaml
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: "3.10"
    - name: Run app.py
      run: python app.py
    ```
    
3. 푸시할 때마다 GitHub Actions가 Python 코드를 실행하게 만들기
    


