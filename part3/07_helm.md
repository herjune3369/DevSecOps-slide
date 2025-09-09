
# 🧑‍🏫 7주차 이론 강의 – Helm 기초

## 1. Helm이란?

- Kubernetes 패키지 매니저
    
- “Kubernetes의 apt/yum, pip, npm 같은 역할”
    
- 여러 개의 YAML 매니페스트 파일을 묶어서 **하나의 패키지(Chart)**로 관리
    
- 반복 배포, 버전 관리, 손쉬운 배포/삭제 가능
    

---

## 2. Helm을 쓰는 이유

- 애플리케이션 배포 시 필요한 매니페스트가 많음 (Deployment, Service, Ingress, ConfigMap 등)
    
- 환경(dev/prod)에 따라 설정이 달라야 함
    
- 수동 관리 → 에러 많음, 반복 작업 불편
    
- Helm Chart로 패키징하면 **한 번에 배포 / 쉽게 업데이트 / 설정값만 바꿔 재사용**
    

---

## 3. Helm 구조

### Chart

- 애플리케이션을 정의한 패키지
    
- 디렉토리 구조:
    
    ```
    mychart/
    ├── Chart.yaml      # 차트 메타데이터
    ├── values.yaml     # 기본 설정값
    ├── templates/      # 쿠버네티스 리소스 매니페스트 템플릿
    ```
    
---
### Release

- 특정 차트를 클러스터에 설치한 인스턴스
    
- 같은 차트를 여러 번 배포 가능 (예: `myapp-dev`, `myapp-prod`)
    

### Repository

- Chart 저장소 (예: ArtifactHub, Bitnami Charts)
    
- `helm repo add`로 저장소 등록 후 설치 가능
    

---

## 4. Helm 주요 파일

### Chart.yaml

- 차트 이름, 버전, 설명 등 메타데이터
    

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
version: 0.1.0
appVersion: "1.0"
```

---
### values.yaml

- 사용자 정의 값 저장 (환경별 설정)
    

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.25"
service:
  type: ClusterIP
  port: 80
```

### templates/

- Kubernetes 매니페스트 템플릿
    
- Go 템플릿 문법 사용
    
- values.yaml의 값을 삽입하여 환경별로 다르게 배포
    

---

## 5. Helm 명령어

- 저장소 추가
    

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

- 차트 검색
    

```bash
helm search repo nginx
```

---

- 설치
    

```bash
helm install my-nginx bitnami/nginx
```

- 릴리스 확인
    

```bash
helm list
```

- 업그레이드
    

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
```

- 삭제
    

```bash
helm uninstall my-nginx
```

---

## 6. Helm의 장점

- **재사용성**: 동일 애플리케이션을 다양한 환경(dev/test/prod)에 쉽게 배포
    
- **유지보수성**: values.yaml만 수정하면 설정 변경 가능
    
- **확장성**: 오픈소스 Chart를 가져와 바로 사용 가능 (DB, 메시지큐, 모니터링 도구 등)
    
- **자동화 친화적**: CI/CD 파이프라인에 통합 용이
    

---

## 7. 활용 예시

- **Nginx 웹서버**: `helm install` 한 줄로 배포
    
- **데이터베이스(MySQL, PostgreSQL)**: Bitnami Chart로 손쉽게 설치
    
- **모니터링 스택(Prometheus + Grafana)**: Helm Chart로 빠르게 구축
    
- **실무**: 대규모 서비스에서 마이크로서비스 수십 개를 Helm Chart로 관리
    

---

## ✅ 정리

- Helm = Kubernetes 패키지 매니저
    
- Chart(패키지) + Values(설정값) + Templates(매니페스트) 구조
    
- 설치, 업그레이드, 삭제 등 라이프사이클 관리 가능
    
- 실무에서 표준적으로 사용되는 **배포 자동화 도구**
    

---

# 💻 7주차 실습 강의 – Helm 기초

## 🎯 실습 목표

- Helm CLI를 설치하고 기본 환경을 준비한다.
    
- Helm Chart 저장소를 등록하고 Nginx를 배포한다.
    
- values.yaml을 수정하여 설정을 변경하고 업그레이드한다.
    
- Helm으로 배포된 애플리케이션을 삭제한다.
    

---

## 1. Helm 설치

### 1-1. 설치 (Linux/Mac)

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 1-2. 버전 확인

```bash
helm version
```

✅ 출력 예시

```
version.BuildInfo{Version:"v3.13.0", GitCommit:"...", GoVersion:"go1.20"}
```

---

## 2. Chart 저장소 등록

### 2-1. Bitnami 저장소 등록

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 2-2. 저장소 업데이트

```bash
helm repo update
```

### 2-3. Chart 검색

```bash
helm search repo nginx
```

✅ `bitnami/nginx` 확인

---

## 3. Nginx 배포

### 3-1. 배포

```bash
helm install my-nginx bitnami/nginx
```

✅ 출력 예시

```
NAME: my-nginx
LAST DEPLOYED: ...
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

---
### 3-2. 리소스 확인

```bash
kubectl get all -l app.kubernetes.io/instance=my-nginx
```

### 3-3. 서비스 접속

```bash
minikube service my-nginx --url
```

✅ 브라우저에서 접속 시 nginx 페이지 표시

---

## 4. values.yaml 수정 후 업그레이드

### 4-1. 기본 values.yaml 다운로드

```bash
helm show values bitnami/nginx > values.yaml
```

### 4-2. replicaCount 수정

```yaml
replicaCount: 3
```
---

### 4-3. 업그레이드 적용

```bash
helm upgrade my-nginx bitnami/nginx -f values.yaml
```

### 4-4. Pod 개수 확인

```bash
kubectl get pods -l app.kubernetes.io/instance=my-nginx
```

✅ Pod가 3개로 늘어남

---

## 5. 릴리스 관리

- 현재 릴리스 목록 확인
    

```bash
helm list
```

- 배포 이력 확인
    

```bash
helm history my-nginx
```

- 삭제
    

```bash
helm uninstall my-nginx
```

✅ 리소스 모두 제거됨

---

## ✅ 최종 결과

- Helm 설치 및 저장소 등록 완료
    
- Nginx Chart를 설치하고 서비스 접속 성공
    
- values.yaml을 수정하여 업그레이드 및 스케일링 실습
    
- Helm 릴리스 삭제까지 라이프사이클 경험
    
