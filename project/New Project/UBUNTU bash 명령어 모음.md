# UBUNTU bash 명령어 모음
<br>
<br>

---

# apt-get 도구 업데이트/업그레이드
```bash
sudo apt-get update (  )
sudo apt-get upgrade ( apt-get 도구 업그레이드 )
```

 gui를 설치하기 전에 apt-get 도구를 update와 upgrade를 진행한다.

---

## ubuntu-desktop 패키지 설치

```bash
sudo apt-get install --no-install-recommends ubuntu-desktop ( 최소 설치 )
sudo apt-get install ubuntu-desktop ( 전체 설치 )
```

- 여타 desktop버전의 프로그램들 ex) 인터넷 브라우저 등 을 설치할 계획이라면, 전체 설치를 하면되고,

DB, jenkins 등 서버용 셋팅만을 원하면 최소 설치를 하기를 권장한다.

---

## gui 설치 후 추가 패키지 설치

```bash
sudo apt-get install indicator-appmenu-tools ( hud service not connected 오류 해결 )

sudo apt-get install indicator-session ( 계정, 세션 아이콘 추가 )

sudo apt-get install indicator-datetime ( 상단 메뉴 시간 추가 )

sudo apt-get install indicator-applet-complete ( 볼륨 조절 아이콘 추가 )
```
 
- gui 패키지 설치 후 발생할 수 있는 `hud service not connected` 오류와 관련하여 indicator-appmenu-tools 
패키지를 통해 해결할 수 있다.
- 나머지 패키지는 사용자의 입장에서 직관적인 편의성을 위한 패지키로써 선택사항입니다.

---

## gui 환경 실행

```bash
startx ( xwindow 환경 실행 )

sudo systemctl isolate graphical.target ( runlevel 5 일회성 실행 / init 실행 )

sudo systemctl enable graphical.target ( runlevel 5 영구히 실행 / 활성 )

sudo systemctl set-default graphical.target ( runlevel 5 영구히 실행 / inittab 수정 )
```
- CLI에 startx  명령어를 입력하면 xwindow 환경이 실행이 되면서 gui 환경으로 전환이 된다.
- `startx` 명령어 없이 영구히 적용하기 위해 위 명령어를 입력하면 된다.

---

# gradle 빌드 / 도커 이미지 빌드 / 복사
```bash
gradlew bootJar
docker build -t anonichat .
docker tag anonichat xotjd794613/anonichat:v0.02
```
- `docker build -t "생성할 이미지 이름" "도커파일을 찾을 위치"`
- `docker tag "복사할 이미지 원본 이름" "복사된 이미지 이름":"태그"`
---

# Docker 다운로드 / 로그인 / DockerHub 이미지 pull
```bash
curl -fsSL https://get.docker.com | sh
docker login
sudo docker pull [image이름]:[태그]
```

---

# 이미지로 컨테이너 실행(일반적)
```bash
sudo docker run -p 8000:8080 "계정명"/"이미지이름":"태그"
```

---

# docker.sock 사용하여 실행(Jenkins용)
```bash
docker run -d \
  --name jenkins-dood \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v jenkins_home:/home/hello \ # 볼륨 마운트 적용
  ghcr.io/anonichat/app/jenkins-dood:v0.06
```
- `--name jenkins-dood \` 실행시킬 컨테이너 이름
- `-p 8080:8080 \` 실행시킬 포트번호
- `xotjd794613/jenkins-dood:v0.01` 실행시킬 이미지 명

---


# root계정으로 접속
```bash
su -
```

---

# 일반계정 docker 그룹으로 묶기
(권한부여 / root계정에서 실행)
``` bash
sudo usermod -aG docker hello(계정명)
```

---

# jenkins admin passwd찾기

```bash
docker exec -it jenkins-dood cat /home/hello/secrets/initialAdminPassword
```

---

# jenkins 버전 확인
```bash
docker exec -it <컨테이너_이름> java -jar /usr/share/jenkins.war --version
```

---

# git(GHCR) docker 컨테이너 로그인
```bash
$  docker login ghcr.io -u "gitHub아이디" 
Password: #<Pesonal Access Token> 입력
```

---

# git(GHCR) docker 태깅
```bash
docker tag "이미지ID" ghcr.io/"gitHub아이디"/"repo이름"/"이미지:태그"
```

---

# DockerCompose 실행 / 종료

```bash
docker-compose up -d
docker-compose down
```

---

# 내부 컨테이너 쉘 접속
```bash
docker exec -it jenkins-dood bash
```

---

# **Server restart 시 컨테이너 Run 모음☘**

```bash
# Jenkins run

docker run -d \
  --name jenkins-dood \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v jenkins_home:/home/hello \
  -v /home/hello/Desktop/AnoniChat/elk-stack:/home/hello/Desktop/AnoniChat/elk-stack \
  ghcr.io/anonichat/app/jenkins-dood:v0.07

---

# elk+Spring run

docker-compose up -d
```

```python

services:
  jenkins:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: jenkins
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./jenkins_home:/home/hello
    networks:
      - data

networks:
  data:
    driver: bridge

```