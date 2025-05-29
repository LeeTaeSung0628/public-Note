# 🤑 ANONI Chat 개발기 - CICD 구성

#프로젝트 #개발 #개요 #구상 #인프라 #CIDE #Jenkins #Vultr

---

<br>

이전 시간에 Clude환경에 docker컨테이너를 구동시키는 것 까지 진행하였다.

▶ [[🤑 ANONI Chat 개발기 - infra setup]]

<br>

이번 시간에는 CICD환경을 구성하겠다.

---

<br>

# <font color="#76923c">Jenkins 연동하기</font>

<br>

## flow Graph

```c
graph TD
	A[GitHub Push]
	B[Jenkins Git Pull]
	C[Gradle Build Jar]
	D[Docker Build & Tag]
	E[DockerHub Push]
	F[서버 Pull + Deploy]
```

- 여기서 해당 방식은 Docker-in-Docker(DinD)방식 중 **DooD** 방식으로 진행한다.

DinD란? - “도커 안에 도커를 실행한다”는 개념.

>[!info] DinD의 종류
>|방식|설명|핵심 개념|
|---|---|---|
|1️⃣ 진짜 DinD (Nested Docker)|컨테이너 안에 Docker 엔진 자체를 설치하고 `dockerd`를 직접 실행|내부에 완전한 Docker daemon|
|2️⃣ Docker-outside-of-Docker (DooD)|컨테이너에서 **호스트의 Docker 데몬에 접근** (`/var/run/docker.sock` 바인드)|외부 Docker 제어|
|3️⃣ Remote Docker|컨테이너에서 **다른 머신의 Docker 데몬**을 TCP로 접근 (`tcp://...`)|원격 제어|

<br>

---

<br>

## Docker에 Jenkins 설치

<br>

일반적인 경우에, Docker에서 Jenkins이미지를 pull받아 컨테이너를 실행시키는 것이 가장 간단하다.
하지만, 이번엔 같은(Ubuntu)서버 내에서 **Jenkins와 Spring서버를 Docker로 함께 띄울**예정 이다.

그렇게 때문에 위에서 기술한 <u>DooD방식으로 Docker로 띄운 Jenkins안에서 Docker를 제어해야한다</u>.

이를위해 `/var/run/docker.sock`을 마운트 해야한다.

*현재 스펙은 ubuntu:22.04 / Spring 3.3.12 / Java17 이다.*

---

<br>

## Jenkins docker image 생성용 dockerfile

```python
# 베이스 이미지 설정
FROM ubuntu:22.04

LABEL maintainer="xotjd794613@naver.com"

#- **비대화식 모드 설정**
# apt 설치 시 발생하는 `timezone 설정`, `Y/N 질문` 등을 자동으로 건너뛰기 위한 설정
ENV DEBIAN_FRONTEND=noninteractive

# 1. UBUNTU 시스템 패키지`s 설치
RUN apt-get update && apt-get install -y \
    curl gnupg2 ca-certificates apt-transport-https software-properties-common \
    git sudo unzip wget lsb-release openjdk-17-jdk \
    && apt-get clean

# 2. 사용자 hello 생성 및 sudo 권한 부여(비밀번호 묻지 않도록)
# Docker CLI 사용이나 기타 시스템 명령 실행 시 필요
RUN useradd -m -d /home/hello -s /bin/bash hello \
    && echo "hello ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

ENV JENKINS_HOME=/home/hello

# 3. Docker CLI 설치 (DooD 방식) - Jenkins 내부에서 `docker build`, `docker run`, `docker push` 명령 사용 가능
# Jenkins가 호스트의 Docker 데몬을 **/var/run/docker.sock**로 제어하게 되는 구조를 전제로 셋팅
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
    > /etc/apt/sources.list.d/docker.list && \
    apt-get update && apt-get install -y docker-ce-cli

# 4. Jenkins WAR 다운로드 (LTS 버전)
# /usr/share/jenkins.war 경로에 배치
ENV JENKINS_VERSION=2.440.1
RUN wget https://get.jenkins.io/war-stable/${JENKINS_VERSION}/jenkins.war -O /usr/share/jenkins.war

# 5. 포트 노출
# → 도커 실행 시 `-p 8080:8080 -p 50000:50000` 으로 외부 연결 가능
EXPOSE 8080
EXPOSE 50000

# 6. Jenkins 실행
# Dockerfile 실행 이후 명령은 `jenkins` 사용자 권한으로 실행됨 (보안을 위해 루트 권한 피함)
USER hello
WORKDIR /var/hello

# `8080` 포트에서 Jenkins 서비스가 시작됨
CMD ["java", "-jar", "/usr/share/jenkins.war"]

```

Docker이미지 빌드
- `jenkins-dood`이름으로 이미지 생성
```c
docker build -t jenkins-dood .
docker tag jenkins-dood xotjd794613/jenkins-dood:v0.01
```

<br> 

push/pull 후 docker.sock 사용하여 실행
```c
docker run -d \
  --name jenkins-dood \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-dood:v0.01

```

> `permission denied` 오류 
![[do-messenger_screenshot_2025-05-28_11_42_51.png]]

즉, **`/var/run/docker.sock` 파일에 접근할 수 있는 권한이 없기 때문에** 발생한 오류이다.
![[do-messenger_screenshot_2025-05-28_12_03_35.png]]

다음과 같이 docker.sock에 접근권한은 root만이 갖고 있기 때문에,
현재 계정을 `docker`그룹에 포함시켜야 한다.

### 해결법

- Dockerfile에 아래 내용을 **추가**하기
```c
# + Docker 그룹id확인 후(GID: 999)에 추가 (호스트와 GID 일치)  
RUN groupadd -g 999 docker && \  
    useradd -m -d /home/hello -s /bin/bash -G docker hello && \  
    echo "hello ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

또는

root계정으로 변경 후 권한주기
``` c
sudo usermod -aG docker hello(계정명)
```


문제 해결 후,
`ip주소:8080`으로 접속하면 jenkins admin 페이지를 확인할 수 있다.

![[Pasted image 20250528142521.png]]


---

<br>


# <font color="#76923c">Jenkins Admin 셋팅</font>


위 페이지에서 admin passwd를 찾기 위해서는 ssh에서 다음 명령어를 통해 알 수 있다.
```bash
docker exec -it jenkins-dood cat /home/hello/secrets/initialAdminPassword
```

- admin 로그인 완료

![[Pasted image 20250528145922.png]]

## 1.  플러그인 설치 화면

<br>

### `Install suggested plugins`

- Jenkins 커뮤니티에서 추천하는 **기본 플러그인 모음**을 자동 설치해줌
- 여기엔 Git, Pipeline, Credentials 등 필수 요소가 포함되어 있음

> !만약 특정한 커스텀 설정이나 최소 설치 환경이 필요한 경우엔 오른쪽을 선택해서 수동으로 선택할 수 있음


>[!failure] 에러
>![[Pasted image 20250528151309.png|625]]
>자동 설치중 대부분에서 `fail`이 발생했다.

### 원인분석

1. ubuntu서버가 외부망에 붙지 못했나??
![[Pasted image 20250528151438.png]]

 *핑 확인시 정상적으로 붙어있는 모습*

그렇다면 무엇이 문제일까?

- 호스트 Ubuntu는 인터넷 연결이 되어 있어도, **Docker 컨테이너 내부**는 DNS나 라우팅 설정이 다를 수 있다.

이를 위한 확인. (컨테이너에 붙기)
```bash
docker exec -it jenkins-dood bash
# 컨테이너에 붙은 후 `apt update로 확인`
```

<br>

![[Pasted image 20250528152038.png|700]]


<br> 

```
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
```

**다음과 같은 로그를 보고 큰 단서를 얻을 수 있었다.**
이 에러는 컨테이너 내부에서 `apt update`를 실행하려 했지만, <u>**현재 사용자가 루트 권한이 아니기 때문에 실패한 것**</u>이다.

jenkins dockerfile 중,,,
다음 부분을 보면 알 수 있다.
```c
# 6. Jenkins 실행
# Dockerfile 실행 이후 명령은 `jenkins` 사용자 권한으로 실행됨 (보안을 위해 루트 권한 피함)
USER hello
WORKDIR /var/hello
```

<br>

<u>하지만...</u>
이미 hello 계정에는 `/etc/sudoers` sudo권한을 준것을 볼 수있다.
```c
RUN groupadd -g 999 docker && \  
    useradd -m -d /home/hello -s /bin/bash -G docker hello && \  
    echo "hello ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

#### 원인을 계속 찾지 못하던 때, Jenkins 버전 up을 통해 해결할 수 있었다.

```java
# /usr/share/jenkins.war 경로에 배치  
#ENV JENKINS_VERSION=2.440.1 플러그인 설치 실패로 인해 버전업  
ENV JENKINS_VERSION=2.462.3  
RUN wget https://get.jenkins.io/war-stable/${JENKINS_VERSION}/jenkins.war -O /usr/share/jenkins.war
```
<br>

**<font color="#76923c">결론</font>**
- 외부망 접근 금지 문제 X
- root/sudo 권한 문제 X
- <u>Jenkins 버전 문제 O</u>

![[do-messenger_screenshot_2025-05-29_11_37_10.png]]

<br>

---

<br>

# <font color="#76923c">Jenkins 파이프라인 구성하기</font>

<br>
