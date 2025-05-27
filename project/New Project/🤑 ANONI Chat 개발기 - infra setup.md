# 🤑 ANONI Chat 개발기 - infra setup

#프로젝트 #개발 #개요 #구상 #인프라 #Vultr

---

<br>

# 초기 인프라 셋팅

<br>

## <font color="#76923c">Spring Boot init</font>


![[Pasted image 20250523150517.png]]
- 스프링 이니셜라이저를 사용하여 java기반의 SpringBoot서버 초기 셋팅

---

<br>

## <font color="#76923c">Vultr</font>

<br>

보안과 확장성을 고려하여 Cloud 서버를 구축하고자 Vultr를 선택하였다.

▶ [[🌩 Cloud-Native Architecture 분석]]

<br>

그 중 Vultr를 선택한 이유는,
1. 성능 대비 가격 효율성
	- AWS, Azure 등 대형 클라우드 서비스보다 저렴하면서, 높은 성능의 인스턴스를 제공한다.
2. 다양한 인스턴스 옵션
	- 공유 CPU / Cloud GPU 등 다양한 유형의 인스턴스를 지원한다.
3. 유연한 요금제와 과금
	- Vultr는 시간 단위로, 사용한 만큼 실시간으로 과금된다.
	- 또한, 포인트를 먼저 결제하고, 그 포인트에서 차감되는 방식으로 초과 과금을 방지하고, 예상하기도 쉽다.
## Vultr Cloude 셋팅

![[do-messenger_screenshot_2025-05-26_11_02_05.png]]
- 서비스 완성 이전까지 월 5달러 스펙의 인스턴스를 띄워놓을 계획이다.
- 추후, 언제나 server스펙을 업그레이드 할 수 있기 때문이다.
- Main ServerOS로는 linux Ubuntu를 선택하였다.

Vultr 인스턴스 생성후, console에서 최소사양으로 ubuntu desktop을 설지하겠다.


---

<br>

 
## <font color="#76923c">ubuntu 초기 셋팅</font>


<br>

```
sudo apt-get update ( apt-get 도구 업데이트 )
sudo apt-get upgrade ( apt-get 도구 업그레이드 )
```

 gui를 설치하기 전에 apt-get 도구를 update와 upgrade를 진행한다.

<br>


## ubuntu-desktop 패키지 설치

```
sudo apt-get install --no-install-recommends ubuntu-desktop ( 최소 설치 )
sudo apt-get install ubuntu-desktop ( 전체 설치 )
```

- 여타 desktop버전의 프로그램들 ex) 인터넷 브라우저 등 을 설치할 계획이라면, 전체 설치를 하면되고,

DB, jenkins 등 서버용 셋팅만을 원하면 최소 설치를 하기를 권장한다.

<br>


## gui 설치 후 추가 패키지 설치

```
sudo apt-get install indicator-appmenu-tools ( hud service not connected 오류 해결 )

sudo apt-get install indicator-session ( 계정, 세션 아이콘 추가 )

sudo apt-get install indicator-datetime ( 상단 메뉴 시간 추가 )

sudo apt-get install indicator-applet-complete ( 볼륨 조절 아이콘 추가 )
```
 
- gui 패키지 설치 후 발생할 수 있는 `hud service not connected` 오류와 관련하여 indicator-appmenu-tools 
패키지를 통해 해결할 수 있다.
- 나머지 패키지는 사용자의 입장에서 직관적인 편의성을 위한 패지키로써 선택사항입니다.

 <br>

## gui 환경 실행

```
startx ( xwindow 환경 실행 )

sudo systemctl isolate graphical.target ( runlevel 5 일회성 실행 / init 실행 )

sudo systemctl enable graphical.target ( runlevel 5 영구히 실행 / 활성 )

sudo systemctl set-default graphical.target ( runlevel 5 영구히 실행 / inittab 수정 )
```
- CLI에 startx  명령어를 입력하면 xwindow 환경이 실행이 되면서 gui 환경으로 전환이 된다.
- `startx` 명령어 없이 영구히 적용하기 위해 위 명령어를 입력하면 된다.
![[do-messenger_screenshot_2025-05-26_11_42_35.png|675]]


>우분투의 runlevel
```c
0 : poweroff.target 
1 : rescue.target 
2, 3, 4 : multi-user.target ( CLI 환경 )
5 : graphical.target  ( GUI 환경 )
6  : reboot.target 
```


---

<br>

## <font color="#76923c">Docker 설정</font>

```python
# 1. 베이스 이미지 (명시적으로 22.04)
FROM ubuntu:22.04  
  
# 2. 작성자 정보  
LABEL maintainer="xotjd794613@naver.com"  
  
# 3. 환경변수 설정 (비인터랙티브 설치)  
ENV DEBIAN_FRONTEND=noninteractive  
  
# 4. 패키지 업데이트 및 JDK 17 설치  
RUN apt-get update && \  
    apt-get install -y openjdk-17-jdk curl && \  
    apt-get clean && rm -rf /var/lib/apt/lists/*
  
# 5. 작업 디렉토리 설정  
WORKDIR /app  
  
# 6. 실행할 jar 파일 복사 (빌드된 .jar)
COPY ./build/libs/AnoniChat-0.0.1-SNAPSHOT.jar /app/AnoniChatApp.jar  
  
# 7. 포트 노출 (SpringBoot default: 8080)
EXPOSE 8080  
  
# 8. 기본 실행 명령  
ENTRYPOINT ["java", "-jar", "/app/AnoniChatApp.jar"]
```
- Spring 프로젝트의 Gradle 빌드 후 `.jar`파일의 위치를 기준으로 docker파일을 작성해주었다.

>[!failure] 이슈
> 다음과 같은 디렉토리 상황에서 
> >
![[do-messenger_screenshot_2025-05-26_17_57_47 1.png]]
> 경로를 찾을 수 없는 오류가 발생하였다.

```
C:\Users\User\Desktop\AnoniChat\
├── App\
│   ├── Dockerfile
│   └── build\
│       └── libs\
│           └── AnoniChat-0.0.1-SNAPSHOT.jar
```

### 원인이 무엇일까?

**.dockerignore** 파일
```
.gradle  
build  
target  
*.iml  
*.log  
.DS_Store  
.git
```

- 이그노어 파일에 build를 추가시켜놓고도 모르고있었다..

<br>


문제 해결 후
- gradle빌드 후 도커 이미지파일 생성
```c
gradlew bootJar
docker build -t anonichat .
```

docker image 생성 완료.

![[Pasted image 20250527095229.png|650]]
![[Pasted image 20250526182326.png]]
- build : .(현재 디렉토리)를 기준으로 빌드하여 이미지 생성
- tag : 현재 anonichat이미지에 태그를 붙여 복사
```c
docker build -t xotjd794613/anonichat:v0.02 .
docker tag anonichat xotjd794613/anonichat:v0.02
docker push "계정명"/anonichat-app:latest
```


- 이미지 생성후 UBUNTU서버에서
1. docker 다운로드
2. docker 로그인
3. image pull 받기
```c
curl -fsSL https://get.docker.com | sh
docker login
sudo docker pull [image이름]:[태그]
```


## <font color="#76923c">실행</font>
```c
sudo docker run -p 8000:8080 "계정명"/anonichat:latest
```
![[Pasted image 20250527103130.png|925]]![[Pasted image 20250527144405.png|575]]