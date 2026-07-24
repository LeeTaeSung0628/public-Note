# 🏧 ATMS - CICD 구성

#프로젝트 #개발 #인프라 #CICD #Jenkins #Docker #GitLab

---

<br>

# <font color="#76923c">프로젝트 소개</font>

<br>

이 프로젝트는 은행 자동화기기(ATM)의 운영 현황을 실시간으로 모니터링하고, 장애를 감지·대응하며, 기기 정보와 운영자 업무를 관리하는 **ATM 통합 관리 시스템(ATMS)** 이다.

Spring Boot 멀티모듈 백엔드 6개(화면 API, 거래저널 수신, 배치/정산, 파일 연동, 대내·대외 인터페이스)와 React 프론트엔드 1개, 총 7개 서비스가 맞물려 운영된다.

<br>

## 왜 CI/CD를 도입했나

배포 대상이 7개나 되다 보니, 처음에는 배포할 때마다 서버에 직접 접속해 `git pull` → 모듈별 재빌드 → 순서대로 재기동을 손으로 반복해야 했다. 이 방식에는 두 가지 문제가 있었다.

1. **모듈이 많을수록 실수할 지점도 늘어난다.** 특정 모듈을 빠뜨리거나 빌드 순서를 헷갈리는 식의 휴먼 에러가 배포마다 발생할 여지가 있었다.
2. **ATM 서비스는 장애 허용치가 낮다.** 배포 실수 하나가 곧바로 기기 상태 모니터링 공백이나 거래 저널 유실로 이어질 수 있는 시스템이라, "사람이 매번 똑같이 실수 없이" 반복하는 방식 자체가 리스크였다.

그래서 `git push` 한 번으로 체크아웃부터 빌드, 재기동까지 **항상 동일한 순서로 실행되는 파이프라인**이 필요하다고 판단했다.

<br>

## AI를 어떻게 활용했나

이번 구축 전 과정에서 **Claude Code를 트러블슈팅 파트너로 적극 활용**했다. 특히 효과를 본 지점은 다음과 같다.

- 에러 로그와 설정 파일을 통째로 넘기고 "가능한 원인 후보"를 먼저 받은 뒤, 하나씩 소거해나가는 순서로 진단 과정을 설계했다.
- 혼자였다면 감으로 이것저것 바꿔보며 시간을 썼을 구간(SSH 인증, CORS)에서, **어느 계층에서 막히고 있는지부터 좁히는 진단 순서**를 먼저 제안받고 검증하는 식으로 진행해 삽질 시간을 크게 줄였다.
- 아래 두 편(SSH Permission Denied, CORS 403)에 그 협업 과정을 그대로 기록해두었다.

---

<br>

# <font color="#76923c">전체 그림 먼저 보기</font>

<br>

```c
graph TD
	A[개발자 git push origin main]
	B[GitLab Webhook]
	C[Jenkins Checkout]
	D[Jenkins → 배포서버 SSH 접속]
	E[서버에서 git pull]
	F[docker compose build]
	G[docker compose up -d]
```

- 별도의 빌드 서버 없이, **GitLab 저장소가 곧 Jenkins의 트리거**가 된다.
- Jenkins는 빌드 자체를 자기 안에서 하지 않고, **SSH로 배포 서버에 직접 접속해 그 안에서 build/up을 실행**시키는 방식을 택했다. (Jenkins는 Checkout까지만 담당)

<br>

>[!info] 컨테이너 구성 (호스트:컨테이너 포트)
>|서비스|포트|비고|
>|---|---|---|
>|frontend|19080:80|React + nginx|
>|atms-web|19081:8080|메인 API|
>|atms-journal|19082:8081|거래저널|
>|atms-batch|19083:8082|배치/통계|
>|atms-file|19084:8083|파일/이미지|
>|atms-if-internal|19085:8084|대내 연계|
>|atms-if-external|19086:8085|대외 연계|
>
> 서버에 이미 다른 서비스가 여럿 떠 있어서, 기본 포트(80/8080...) 대신 19000번대로 전부 옮겼다. *(기존 서비스와 포트가 겹쳐 한 번 충돌을 겪은 뒤 정착한 규칙이다.)*

---

<br>

# <font color="#76923c">Jenkins, 컨테이너로 띄우기</font>

<br>

Jenkins도 그냥 설치하지 않고 **커스텀 이미지**로 띄웠다. 이유는 두 가지다.

1. Jenkins가 호스트의 Docker를 직접 제어해야 한다 (`docker compose build/up`을 실행해야 하므로) → 이미지 안에 **Docker CLI**가 있어야 한다.
2. 플러그인 설치나 초기 보안 설정을 매번 수동으로 클릭하고 싶지 않았다 → **JCasC(Jenkins Configuration as Code)** 로 자동화했다.

```dockerfile
FROM jenkins/jenkins:lts-jdk21
USER root
# Docker CLI 설치 — Jenkins가 호스트 Docker를 제어할 수 있도록
RUN apt-get update && apt-get install -y ca-certificates curl gnupg lsb-release && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
      https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && apt-get install -y docker-ce-cli && rm -rf /var/lib/apt/lists/*
RUN groupadd -f docker && usermod -aG docker jenkins
USER jenkins
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
COPY casc/jenkins.yaml /var/jenkins_casc/jenkins.yaml
ENV CASC_JENKINS_CONFIG=/var/jenkins_casc/jenkins.yaml
```

>[!warning] ssh-agent 플러그인을 빼먹지 말 것
> `plugins.txt`에 아래 플러그인들, 특히 **`ssh-agent`** 를 반드시 넣어야 한다.
> ```text
> workflow-aggregator
> git
> gitlab-plugin
> credentials-binding
> ssh-agent      # ← Jenkinsfile의 sshagent() 스텝에 필수
> ws-cleanup
> configuration-as-code
> ```
> 이걸 빼먹으면 나중에 **Deploy 단계가 로그 한 줄 없이 조용히 실패**한다. (다음 편에서 다룰 이야기의 복선이다.)

<br>

JCasC 설정은 최대한 단순하게 유지했다. 처음에 `local`과 `gitLab` 인증 방식을 동시에 선언했다가 Jenkins가 **무한 재시작 루프**에 빠진 적이 있는데, 인증 방식을 하나만 남기고 나서야 정상 기동했다.

```yaml
jenkins:
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "${JENKINS_ADMIN_ID}"
          password: "${JENKINS_ADMIN_PASSWORD}"
  authorizationStrategy:
    loggedInUsersCanDoAnything:
      allowAnonymousRead: false
```

---

<br>

# <font color="#76923c">두 개의 인증, 절대 헷갈리면 안 된다</font>

<br>

이번 구축에서 Jenkins는 **서로 다른 두 곳**에 접속하고, 각각 **완전히 다른 인증 수단**을 쓴다. 여기서 한 번 헷갈렸다가 뒤에서 크게 고생했다.

```
① Jenkins → GitLab (소스 코드 받기)
     인증수단: GitLab Deploy Token (아이디/비밀번호 형태)

② Jenkins → 배포 서버 (실제 배포 실행)
     인증수단: SSH 개인키 (열쇠 형태)
```

>[!question] GitLab 토큰 하나로 서버 SSH까지 되지 않나?
> 안 된다. Deploy Token은 **GitLab 저장소 읽기 전용** 권한일 뿐, 서버 SSH 접속과는 완전히 별개의 인증 체계다. 이걸 같은 것으로 착각하면 반드시 막힌다.

GitLab 쪽은 개인 계정에 종속되지 않는 **Deploy Token**(읽기 전용)을 발급해서 썼다. Personal Access Token은 계정이 삭제되거나 비밀번호가 바뀌면 CI가 통째로 멈추기 때문에 처음부터 후보에서 제외했다.

---

<br>

# <font color="#76923c">Jenkinsfile 작성</font>

<br>

최종적으로 완성한 Jenkinsfile은 다음과 같다. Checkout과 Deploy, 딱 두 스테이지뿐이다.

```groovy
pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DEPLOY_HOST = '10.0.20.15'
        DEPLOY_USER = 'deploy'
        DEPLOY_DIR  = '/home/deploy/atms'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['atms-server-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no \\
                            -o ConnectTimeout=10 \\
                            ${DEPLOY_USER}@${DEPLOY_HOST} '
                            cd ${DEPLOY_DIR}
                            git pull origin main
                            docker compose build
                            docker compose up -d --remove-orphans
                        '
                    """
                }
            }
        }
    }

    post {
        success { echo "✅ 배포 성공 | 커밋: ${env.GIT_COMMIT_SHORT}" }
        failure { echo "❌ 배포 실패 | Jenkins 콘솔 로그를 확인하세요" }
        always  { cleanWs() }
    }
}
```

- `disableConcurrentBuilds()` : 배포 도중 또 다른 배포가 겹쳐 서비스가 불안정해지는 걸 막는다.
- `sshagent(credentials: [...])` 블록 안에서만 SSH 개인키가 살아있고, 블록을 벗어나면 자동으로 해제된다.
- GitLab Webhook과 Jenkins Job의 **Secret Token**을 맞춰주는 것도 잊지 말아야 한다. 안 그러면 Webhook 테스트에서 `anonymous is missing the Job/Build permission` 이라는 403을 받게 된다.

---

<br>

여기까지 구성하고 나니, 파이프라인 자체는 그럴듯하게 완성된 것처럼 보였다.

>[!tip] 이 지점부터는 Claude Code와 함께 원인을 추적했다
> 앞서 적었듯, 증상을 그대로 AI에게 던지고 **가설을 세우고 → 검증하고 → 다음 가설로 넘어가는** 과정을 함께 진행했다. 다음 두 편은 그 과정을 그대로 기록한 글이다.

하지만 실제로 `git push`를 날려보자, 파이프라인은 곧바로 **두 개의 벽**에 부딪혔다.

# 다음에는, 첫 번째 벽이었던 **SSH Permission Denied**를 다뤄보겠다.

## ▶ [[🏧 ATMS - CICD 트러블슈팅(SSH Permission Denied편)]]
