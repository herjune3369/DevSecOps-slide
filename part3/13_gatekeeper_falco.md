# 🧑‍🏫 13주차 이론 강의 – Gatekeeper 정책 위반 탐지와 Falco 런타임 보안 이벤트 분석

## 1. Kubernetes 보안의 두 가지 축

- **정책 기반 보안(Policy Enforcement)**
    
    - 클러스터에 배포되는 리소스(YAML) 검증
        
    - 잘못된 설정이나 보안 위반을 **사전에 차단**
        
- **런타임 보안(Runtime Security)**
    
    - 실행 중인 컨테이너/Pod 모니터링
        
    - 비정상 행위(예: 쉘 실행, 민감 파일 접근)를 **실시간 탐지**
        

👉 Gatekeeper = **정책 위반 사전 차단**  
👉 Falco = **런타임 행위 실시간 탐지**

---

## 2. Gatekeeper 개요

- Open Policy Agent(OPA) 기반 Kubernetes 정책 엔진
    
- **Admission Controller** 역할 → 리소스 생성 요청 시 정책 검증
    
- 주요 개념
    
    - **ConstraintTemplate**: 정책 템플릿 (Rego 언어 기반)
        
    - **Constraint**: 특정 정책 정의 (예: 이미지 레지스트리 제한)
        
- 예시 정책
    
    - 컨테이너는 반드시 특정 레지스트리에서만 가져오기
        
    - 모든 Pod는 `requests/limits` 리소스 설정 필수
        
    - Privileged 모드 금지
        

---

## 3. Gatekeeper 동작 흐름

1. 사용자가 `kubectl apply -f pod.yaml` 실행
    
2. API Server → Admission Controller(Gatekeeper)에 전달
    
3. Gatekeeper가 Constraint와 비교
    
4. 위반 시 → 배포 거부 (`admission webhook "validation.gatekeeper.sh" denied the request`)
    
5. 정상 시 → etcd 저장 → Pod 생성
    

---

## 4. Falco 개요

- CNCF 프로젝트, **런타임 보안 모니터링 툴**
    
- Linux 커널(eBPF/Syscall) 기반으로 시스템 호출 추적
    
- 컨테이너에서 발생하는 **비정상 행위** 탐지
    
- 기본 제공 룰(rule) 예시
    
    - 컨테이너에서 `bash` 실행 시 경고
        
    - `/etc/passwd` 파일 접근 시 경고
        
    - 민감한 네트워크 포트 열림 탐지
        

---

## 5. Falco 동작 흐름

1. 컨테이너 실행 중 이벤트 발생 (예: `bash` 실행)
    
2. Falco 에이전트가 Syscall(eBPF) 캡처
    
3. 룰과 비교 → 위반 시 이벤트 생성
    
4. 경고 로그 출력 (stdout, 파일, Syslog, Prometheus 등)
    
5. 보안팀이 알림 확인 → 대응 조치
    

---

## 6. Gatekeeper vs Falco 비교

|구분|Gatekeeper (OPA)|Falco (CNCF)|
|---|---|---|
|보안 단계|사전 예방 (Prevention)|실행 중 탐지 (Detection)|
|적용 방식|Admission Controller|커널 레벨 이벤트 모니터링|
|정책 예시|특정 레지스트리만 허용|컨테이너 내 쉘 실행 탐지|
|활용 목적|잘못된 배포 차단|이상 행위 실시간 경고|

---

## 7. 실무 활용 예시

- Gatekeeper:
    
    - “모든 Pod는 반드시 `runAsNonRoot` 설정 포함”
        
    - “imagePullPolicy는 Always로 설정해야 함”
        
- Falco:
    
    - “컨테이너에서 `/bin/sh` 실행 감지”
        
    - “권한 없는 사용자가 root 디렉토리 접근 시 알림”
        

---

## ✅ 정리

- **Gatekeeper**: 쿠버네티스 리소스 배포 시 정책 위반 차단 (OPA 기반)
    
- **Falco**: 실행 중 컨테이너에서 비정상 행위 탐지 (Syscall/eBPF 기반)
    
- 두 도구를 함께 사용하면 DevSecOps에서 **사전 예방 + 실시간 탐지** 보안 체계를 구축할 수 있음
    
- 다음(13주차 실습)에서는 Gatekeeper 설치 후 정책 위반 테스트, Falco 설치 후 런타임 이벤트 탐지를 실습
    

---

# 💻 13주차 실습 강의 – Gatekeeper 정책 위반 탐지와 Falco 런타임 보안 이벤트 분석

## 🎯 실습 목표

- Gatekeeper를 설치하고 정책(Constraint)을 적용한다.
    
- 잘못된 매니페스트를 배포하여 정책 위반 차단을 확인한다.
    
- Falco를 설치하여 컨테이너 런타임 이벤트를 탐지한다.
    
- 정상/비정상 행위에 대한 로그를 확인한다.
    

---

## 1. Gatekeeper 실습

### 1-1. Gatekeeper 설치

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.15/deploy/gatekeeper.yaml
```

설치 확인:

```bash
kubectl get pods -n gatekeeper-system
```

✅ `gatekeeper-controller-manager` Pod Running 확인

---

<!-- _class: medium -->

### 1-2. 정책 템플릿 생성 (Privileged 금지)

`constrainttemplate.yaml`

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8spspprivilegedcontainer
spec:
  crd:
    spec:
      names:
        kind: K8sPSPPrivilegedContainer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8spspprivilegedcontainer

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          container.securityContext.privileged == true
          msg := sprintf("Privileged container is not allowed: %v", [container.name])
        }
```

적용:

```bash
kubectl apply -f constrainttemplate.yaml
```

---

### 1-3. Constraint 생성

`constraint.yaml`

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPPrivilegedContainer
metadata:
  name: disallow-privileged
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
```

적용:

```bash
kubectl apply -f constraint.yaml
```

---

### 1-4. 정책 위반 Pod 생성 시도

`privileged-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
```

배포 시도:

```bash
kubectl apply -f privileged-pod.yaml
```

✅ 출력 예시

```
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [disallow-privileged] Privileged container is not allowed: nginx
```

---

## 2. Falco 실습

### 2-1. Helm 저장소 추가 및 Falco 설치

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco -n falco --create-namespace
```

설치 확인:

```bash
kubectl get pods -n falco
```

✅ `falco-xxxx` Pod Running 확인

---

### 2-2. Falco 로그 확인

```bash
kubectl logs -n falco -l app.kubernetes.io/name=falco
```

---

### 2-3. 비정상 행위 유발 (컨테이너 안에서 쉘 실행)

1. 테스트 Pod 생성
    

```bash
kubectl run test --image=nginx -- sleep 3600
```

2. 컨테이너 접속 시도
    

```bash
kubectl exec -it test -- /bin/bash
```

3. Falco 로그 확인
    

```bash
kubectl logs -n falco -l app.kubernetes.io/name=falco | grep "bash"
```

✅ 출력 예시:

```
22:10:01.123456 Warning Sensitive Mount: Pod (test) tried to spawn shell inside container (bash)
```

---

## 3. 정리

### 리소스 삭제

```bash
kubectl delete -f privileged-pod.yaml
kubectl delete -f constraint.yaml
kubectl delete -f constrainttemplate.yaml
helm uninstall falco -n falco
kubectl delete ns falco
kubectl delete ns gatekeeper-system
```

---

## ✅ 최종 결과

- Gatekeeper 설치 및 정책 적용 성공 → 보안 위반 Pod 차단 확인
    
- Falco 설치 후 런타임 이벤트 감지 성공 → 컨테이너 내 bash 실행 탐지
    
- 학생들은 **정책 위반 사전 차단(Gatekeeper)** + **실시간 행위 탐지(Falco)**의 보안 체계를 경험
    

