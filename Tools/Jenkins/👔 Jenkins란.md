# 👔 Jenkins란

#Tools #도커 #Docker

---

# <font color="#9bbb59">젠킨스(Jenkins)란 무엇이고, 언제 사용하는걸까?</font>

- **Jenkins**는 오픈소스 기반의 **자동화 서버**로, 주로 **지속적 통합**(CI: Continuous Integration)과 **지속적 배포**(CD: Continuous Delivery/Deployment)를 자동화하기 위해 사용된다.

특징
- java기반으로 만들어져, OS에 상관없이 실행 가능
- 파이프라인 기반으로 소프트웨어 빌드, 테스트, 배포 자동화
- 플러그인 아키텍처로 확장성이 뛰어남

---

# 젠킨스의 핵심 역할
### 🛠 1) **지속적 통합 (CI)**

- 개발자가 코드 변경 시 자동으로 빌드 + 테스트 수행
- 코드 병합 시 문제 조기 탐지
- 코드 품질 유지 + 릴리즈 주기 단축

### 📦 2) **지속적 배포/배포 (CD)**

- 테스트 통과 후 자동으로 **스테이징 또는 운영 서버**에 배포
- 릴리즈 주기 자동화 → 빠른 피드백 루프

### 🔄 3) **자동화 워크플로우 관리**

- 빌드 → 테스트 → 배포 → 모니터링까지의 전체 과정을 파이프라인으로 시각화
![[Pasted image 20250421164348.png|750]]
- Git Hook, Cron, Webhook, Polling 등 다양한 트리거 지원

>[!hint] *Webhook* vs *Polling*
>
> 폴링(Poliing) : 일정 시간 간격으로 저장소의 변경사항을 확인
> 웹훅(Webhook) : 저장소(Git 등)에 변경사항이 발생했을 때, 젠킨스에 즉시 알려주는 방식

<u>이를 설정하는 방법은 젠킨스의 파이프 라인 설정을 통해 이루어 진다.</u>

- 파이프라인 구조 예시
```Groovy
pipeline {
    agent any
    triggers {
        // GitHub에서 Webhook을 통해 이벤트가 올 때 트리거됨
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/your-org/your-repo.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'echo Building...'
            }
        }
    }

----

    agent any
    triggers {
        // 5분마다 Git 변경사항 확인 (cron 표현식)
        pollSCM('H/5 * * * *')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/your-org/your-repo.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'echo Building...'
            }
        }
    }
}
```

---

# 젠킨슨의 주요 기능
### 🧱 1) **Job과 Build System**

- `Freestyle Job`: 기본 작업 단위 (빌드, 테스트, 배포 설정)
- `Pipeline`: Groovy 기반 DSL로 멀티스텝 작업 정의
- `Multibranch Pipeline`: 브랜치마다 파이프라인 분기 설정

### 🔌 2) **Plugin 생태계**

- 1,800개 이상의 공식 플러그인
- 주요 플러그인:
    - Git / GitHub / Bitbucket
    - Docker
    - Slack / Email Notification
    - Kubernetes
    - SonarQube
    - Artifactory / Nexus

### 💬 3) **Notification 및 알림 기능**

- 이메일, Slack, Teams, Webhook 등 다양한 알림 지원
- 실패/성공/경고 상태에 따라 조건부 메시지 전송 가능

### 📊 4) **빌드 이력 및 로깅**

- 작업당 빌드 결과 로그 저장
- 실패 시 어떤 스텝에서 실패했는지 명확히 추적 가능

### 🔒 5) **사용자 및 권한 관리**

- LDAP, Active Directory 연동
- Role 기반 접근 제어 (RBAC)
- Job/Folder 단위 권한 설정 

### ☁️ 6) **분산 빌드 (Master-Agent 아키텍처)**

- Master는 Job 스케줄링 및 관리
- Agent (노드)는 실제 빌드 실행 담당
- 스케일아웃 구조로 부하 분산 가능

---

# Jenkins와 다른 CI/CD 도구 비교

|항목|Jenkins|GitHub Actions|GitLab CI|CircleCI|
|---|---|---|---|---|
|설치 방식|자체 설치 (on-prem, cloud)|클라우드 기반|자체/클라우드|클라우드 중심|
|파이프라인 언어|Groovy (Jenkinsfile)|YAML|YAML|YAML|
|확장성|매우 높음 (플러그인 중심)|GitHub 연계 중심|GitLab과 밀접|빠른 CI 특화|
|러닝 커브|중간~높음|쉬움|중간|쉬움|

---

# <font color="#9bbb59">Jenkins와 Docker</font>

## 도커에 대한 설명 ▶[[🐋 docker 란]]

젠킨스와 도커는 서버 자동화(CI/CD)환경에서 매우 자주 함께 활용된다.
두 도구의 역할은 서로 다르지만 서로를 보완해주는 역할을 한다.

먼저 도커의 주 역할은 **애플리케이션 실행 환경의 캡슐화** 이다.
이렇게 캡슐화된 것을 `이미지` 또는 `컨테이너`라고 한다.

>[!warning] 이미지와 컨테이너?
> 이미지와 컨테이너는 사실상 같은 개념을 가리키는 용어이다.
> 
> 도커 이미지 : Docker라는 특정 기술/플랫폼을 통해 생성되고 관리되는 이미지. Docker에 종속적
> 컨테이너 : 보다 일반적인 용어로, 컨테이너 기술을 사용하는 모든 플랫폼의 이미지를 뜻한다.

즉, 젠킨스와 도커의 관계를 요약하자면
Jenkins는 Docker를 사용해 **컨테이너 환경에서 빌드와 테스트를 실행하거나**, **애플리케이션을 Docker 이미지로 빌드하고 레지스트리에 배포**하며, 필요 시 Jenkins 자체도 Docker 컨테이너로 실행할 수 있다.

### 실행 플로우

1. **파이프라인 실행 트리거**
    - GitHub/GitLab에서 코드 변경 발생 시:
        - ✅ **Webhook**: 변경 사항을 Jenkins에 즉시 알림
        - ✅ **Polling**: Jenkins가 주기적으로 변경 여부 확인
            
2. **소스코드 + Dockerfile 가져오기**
    - 트리거 감지 후 Jenkins가 Git 저장소에서 코드와 `Dockerfile`을 `checkout

>[!hint] dockerfile
> → 컨테이너 이미지를 만들기 위한 **설정 파일(스크립트)**
 쉽게 말해, **“어떤 OS에, 어떤 앱을 설치하고, 어떤 명령을 실행할 것인지”를 정의한 레시피**라고 볼 수 있다.

3. **Docker 이미지 빌드**
    - `docker build -t my-app .` 명령으로 컨테이너 이미지 생성
        
4. **컨테이너 기반 테스트 수행 (선택)**
    - `docker run`으로 테스트 자동화 실행
        
5. **Docker 이미지 푸시**
    - DockerHub, ECR 등에 `docker push` 수행
        
6. **서버 또는 클러스터에 배포**
    - SSH, Docker Compose, <u>Kubernetes</u> 등을 통해 자동 배포
        ▶ [[🚢Kubernetes(k8s)란]]
7. **알림 및 로그 저장**
    - Slack, Email 알림 전송 + Jenkins에 빌드 로그 기록

