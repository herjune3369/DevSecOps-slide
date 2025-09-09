
# 🧑‍🏫 2주차 이론 강의 – Docker 컨테이너 기초

## 1. 컨테이너의 필요성

- 과거 개발 환경의 문제: “내 PC에서는 되는데 서버에서는 안 된다”
    
- VM(가상머신)의 한계: 무겁고 느림, 자원 낭비 심함
    
- 컨테이너: 애플리케이션과 실행 환경을 함께 패키징 → 어디서나 동일 실행
    
- 장점: 가볍고 빠르며 이식성이 뛰어남
    

---

## 2. Docker란?

- 2013년 등장, 컨테이너 활용을 대중화시킨 오픈소스 플랫폼
    
- 개발부터 배포까지 표준화된 프로세스 제공
    
- 핵심 기능: 이미지 생성, 컨테이너 실행, 자원 관리, 공유(Registry)
    

---

## 3. Docker 아키텍처

- **클라이언트(docker CLI)**: 명령어 입력
    
- **데몬(Docker Daemon)**: 컨테이너 관리, API 처리
    
- **이미지(Image)**: 실행 가능한 애플리케이션 패키지
    
- **컨테이너(Container)**: 이미지를 실행한 상태
    
- **레지스트리(Registry)**: 이미지 저장소 (Docker Hub, AWS ECR 등)
    

---

## 4. VM vs 컨테이너

- **VM**: 하이퍼바이저 위에 OS를 올려 실행, 자원 소모 큼
    
- **컨테이너**: 호스트 OS 커널을 공유, 훨씬 가벼움
    
- 비교 정리:
    
    - VM: 느림, 무거움, 보안성 높음
        
    - 컨테이너: 빠름, 가벼움, 이식성 뛰어남
        

---

## 5. Docker 핵심 개념

- **Dockerfile**: 이미지를 정의하는 스크립트 (베이스 이미지, 실행 명령어 포함)
    
- **Image**: 실행 환경과 코드가 포함된 정적 패키지
    
- **Container**: 이미지를 실행한 동적 인스턴스
    
- **Registry**: 이미지를 저장하고 공유하는 공간
    

---

## 6. 데이터 관리와 네트워크

- **Volume**: 데이터 영속화, 컨테이너 종료 후에도 데이터 유지
    
- **Network**: 여러 컨테이너를 하나의 가상 네트워크로 묶어 통신 가능
    

---

## 7. Docker의 장점

- 실행 속도: VM보다 수십 배 빠른 시작
    
- 효율성: 동일한 서버에서 더 많은 애플리케이션 실행
    
- 이식성: 개발 환경 = 운영 환경, 일관성 보장
    
- DevOps 친화적: CI/CD 파이프라인과 자연스럽게 연계
    

---

## 8. Docker의 한계

- 개별 컨테이너 관리에는 적합하지만, 대규모 서비스 운영에는 한계
    
- 자동 확장, 서비스 디스커버리, 장애 복구 기능 부족
    
- → Kubernetes 같은 오케스트레이션 도구 필요
    

---

## ✅ 정리

- Docker는 컨테이너 실행의 표준 플랫폼
    
- VM 대비 가볍고 빠르며, 클라우드 네이티브 환경의 기반 기술
    
- 다음 주차(3주차)에서는 Kubernetes 아키텍처로 확장
    


---

# 💻 2주차 실습 강의 – Docker 컨테이너 기초

## 🎯 실습 목표

- Docker를 설치하고 동작을 확인한다.
    
- 컨테이너를 실행하고 외부에서 접속한다.
    
- 여러 컨테이너를 실행하고 관리한다.
    
- Volume과 Network 기능을 간단히 체험한다.
    

---

## 1. Docker 설치 및 확인

### 1-1. 설치 (이미 설치되어 있으면 건너뛰기)

- [Docker Desktop](https://docs.docker.com/get-docker/) 다운로드 후 설치
    
- Windows → WSL2 필요, Mac → Intel/M1 선택
    

### 1-2. 설치 확인

```bash
docker --version
```

✅ 정상 출력 예시:

```
Docker version 24.0.5, build ced0996
```

---



## 2. 첫 번째 컨테이너 실행

### 2-1. nginx 이미지 다운로드

```bash
docker pull nginx
```

✅ 출력: `Status: Downloaded newer image for nginx:latest`

### 2-2. 컨테이너 실행

```bash
docker run -d -p 8080:80 --name webserver nginx
```

- `-d` : 백그라운드 실행
    
- `-p 8080:80` : 호스트 8080 → 컨테이너 80 포트 매핑
    
- `--name webserver` : 컨테이너 이름 설정
    

---

### 2-3. 실행 확인

```bash
docker ps
```

✅ 출력 예시:

```
CONTAINER ID   IMAGE   COMMAND                  STATUS         PORTS                  NAMES
abc12345       nginx   "nginx -g 'daemon of…"   Up 2 minutes   0.0.0.0:8080->80/tcp   webserver
```

### 2-4. 브라우저 접속

- 주소창에 `http://localhost:8080` 입력
    
- ✅ "Welcome to nginx!" 페이지 확인
    

---

## 3. 컨테이너 관리

### 3-1. 로그 확인

```bash
docker logs webserver
```

### 3-2. 컨테이너 접속

```bash
docker exec -it webserver /bin/bash
```

✅ 컨테이너 내부로 들어가면 Linux 쉘 환경 확인 가능

### 3-3. 컨테이너 중지 & 삭제

```bash
docker stop webserver
docker rm webserver
```

---

## 4. Volume 체험 (데이터 유지)

### 4-1. HTML 페이지 저장용 디렉토리 생성

```bash
mkdir ~/nginx-html
echo "Hello Docker Volume" > ~/nginx-html/index.html
```

### 4-2. 컨테이너 실행 (볼륨 마운트)

```bash
docker run -d -p 8081:80 \
  -v ~/nginx-html:/usr/share/nginx/html \
  --name webserver2 nginx
```

### 4-3. 접속 확인

- 브라우저에서 `http://localhost:8081`
    
- ✅ "Hello Docker Volume" 출력
    

---

## 5. Network 체험 (컨테이너 간 통신)

### 5-1. 네트워크 생성

```bash
docker network create mynet
```

### 5-2. 컨테이너 두 개 실행

```bash
docker run -d --network=mynet --name nginx1 nginx
docker run -d --network=mynet --name nginx2 nginx
```

### 5-3. 네트워크 확인

```bash
docker network inspect mynet
```

✅ `nginx1`, `nginx2` 두 컨테이너가 같은 네트워크에 있음 확인

---

## 6. 정리 (클린업)

### 6-1. 실행 중인 모든 컨테이너 종료 및 삭제

```bash
docker ps -q | xargs docker stop
docker ps -aq | xargs docker rm
```

### 6-2. 생성한 네트워크 삭제

```bash
docker network rm mynet
```

---

## ✅ 최종 결과

- 학생들은 Docker 컨테이너를 직접 실행하고 웹 브라우저에서 서비스 확인
    
- Volume과 Network 기능을 간단히 실습
    
- 다음 주차(3주차) Kubernetes 아키텍처 학습으로 자연스럽게 연결
    

