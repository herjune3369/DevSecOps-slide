
---
# 3주차: CWPP 이론 – 컨테이너 워크로드 보안

---

## 🎯 학습 목표

- CWPP 개념과 필요성 이해
    
- 컨테이너 보안 위협 유형 설명
    
- CWPP 도구의 기능과 한계 파악
    

---

## 1️⃣ CWPP 개요

- **CWPP (Cloud Workload Protection Platform)**
    
    - 컨테이너, VM, 서버리스 워크로드 보호
        
    - 취약점 탐지, 런타임 보호, 규정 준수 지원
        
- **DevSecOps에서의 역할**
    
    - CSPM: 클라우드 계정·IaC 점검
        
    - CWPP: 워크로드 점검
        
    - CNAPP: CSPM + CWPP 통합
        

---

## 2️⃣ 컨테이너 보안 위협 유형

- **이미지 취약점**
    
    - 오래된 베이스 이미지, 패키지 결함
        
- **잘못된 설정**
    
    - root 실행, 포트 과다 노출, 환경변수에 민감정보 저장
        
- **런타임 위협**
    
    - 컨테이너 탈출(Container Escape)
        
    - 악성 프로세스, 크립토마이닝
        

---

## 3️⃣ CWPP 솔루션 주요 기능

1. **취약점 점검**: 이미지·패키지 CVE 탐지
    
2. **런타임 보호**: 비정상 행위 탐지 및 차단
    
3. **규정 준수 검사**: CIS, NIST, PCI-DSS 등 표준 준수
    
4. **리포트/대시보드**: 위험도, 우선순위 제공
    

---

## 4️⃣ Trivy와 CWPP

- Trivy: **취약점 점검**에 특화
    
- 장점
    
    - 설치 간단, 가볍고 빠름
        
    - 다양한 출력(JSON, SARIF) 지원
        
- 한계
    
    - 런타임 보호 미흡
        
    - False Positive 존재
        

---

## 5️⃣ 산업 적용 사례

- **금융권**: 배포 전 이미지 스캔 → PCI-DSS 준수
    
- **스타트업**: Kubernetes Pod 이미지 점검 자동화
    
- **대기업**: CNAPP 솔루션 도입해 전사 보안 관리
    

---

## 6️⃣ 토론 과제

1. CSPM과 CWPP의 차이는 무엇인가?
    
2. 단순 이미지 취약점 점검만으로 충분하지 않은 이유는?
    
3. 운영 환경에서 CWPP가 제공해야 할 핵심 기능은 무엇인가?
    

---

## ✅ 정리

- CWPP는 컨테이너 및 워크로드 보안 전담
    
- 위협은 이미지 취약점, 잘못된 설정, 런타임 위협으로 구분
    
- Trivy는 CWPP의 취약점 점검 영역을 담당
    
- CSPM과 함께 CNAPP의 핵심 축을 이룸

---
# 3주차: CWPP – 컨테이너 이미지 취약점 점검 (실습, 3시간)

---

## 🎯 학습 목표

- CWPP 개념을 이해한다
    
- Trivy를 활용해 컨테이너 이미지 취약점을 점검한다
    
- 결과를 해석하고 개선 방안을 제시한다
    

---

## 1️⃣ CWPP 개요 (20분)

- **CWPP (Cloud Workload Protection Platform)**
    
    - 컨테이너, VM, 서버리스 워크로드 보호
        
    - 취약점 탐지, 런타임 보호, 규정 준수 기능 제공
        
- **DevSecOps 내 역할**
    
    - CSPM: IaC/클라우드 설정 점검
        
    - CWPP: 실행 워크로드 점검
        
    - CNAPP: CSPM + CWPP 통합
        

---

## 2️⃣ 실습 환경 준비 (40분)

### Docker 이미지 준비

```bash
docker pull python:3.9
docker pull nginx:latest
```

### Trivy 설치 확인

```bash
trivy --version
```

👉 예상 출력

```
Version: 0.50.0
Vulnerability DB: 2025-09-05
```

---

## 3️⃣ Python 이미지 스캔 (30분)

```bash
trivy image python:3.9
```

👉 예상 출력

```
python:3.9 (debian 11.7)
Total: 12 (LOW: 3, MEDIUM: 5, HIGH: 3, CRITICAL: 1)
```

- 주요 취약점: OpenSSL, libssl 등
    
- CVE ID, 심각도, 패치 필요 버전 표시
    

**확장 미션**

- `python:3.8`, `python:3.11` 스캔 → 버전별 취약점 비교
    

---

## 4️⃣ Nginx 이미지 스캔 (30분)

```bash
trivy image nginx:latest
```

👉 예상 출력

```
nginx:1.25 (debian 11.7)
Total: 8 (LOW: 2, MEDIUM: 3, HIGH: 2, CRITICAL: 1)
```

- Python: 라이브러리 취약점 多
    
- Nginx: 웹 서버 구성요소 취약점
    

**확장 미션**

- `nginx:stable` vs `nginx:alpine` 비교 → 어떤 태그가 더 안전한가?
    

---

## 5️⃣ JSON 결과 저장 및 분석 (30분)

```bash
mkdir -p reports
trivy image -f json -o reports/python.json python:3.9
trivy image -f json -o reports/nginx.json nginx:latest
```

👉 JSON 파일에서 취약점 세부 정보 확인

```bash
cat reports/python.json | jq '.Results[0].Vulnerabilities[0]'
```

**추가 과제**

- Critical 취약점만 따로 추출해 표로 정리
    

---

## 6️⃣ 팀별 실습 (20분)

1. 팀별로 다른 이미지(`redis`, `mysql`, `ubuntu`) 선택 후 스캔
    
2. JSON 결과 저장 후 Repo 업로드
    
3. Critical/High 취약점 1개 골라 원인 분석 및 패치 방법 조사
    

---

## 7️⃣ 발표 & 토론 (20분)

- 팀별 결과 발표 (5분씩)
    
- 공통 취약점 및 특이 취약점 공유
    
- 토론: 운영 환경에서 **어떤 이미지 태그 전략**이 안전한가?
    

---

## ✅ 정리

- CWPP는 실행 워크로드 보안 점검을 담당
    
- Trivy `trivy image`로 컨테이너 취약점 탐지 가능
    
- 버전별 이미지 비교 → 최신/경량 버전 선택이 중요
    
- JSON 결과는 자동화 및 리포트 활용 가능
    

👉 다음 주(4주차 실습): **CSPM – IaC 보안 점검 (`trivy config ./terraform`)**

---
