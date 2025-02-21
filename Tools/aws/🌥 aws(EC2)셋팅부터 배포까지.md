# 🌥 aws(EC2)셋팅부터 배포까지

#Tools #AWS #Amazon #Cloud #Docker #Redis 

---
![[Pasted image 20250221112429.png]]
### AWS의 EC2란 무엇인가?

- AWS가 제공하는 **'클라우드 컴퓨팅 서비스'** 이다.

예전의 쓴 더 자세한 글.
# → **여기로** [[🍊 aws EC2란]]
# → **여기로** [[📘 SpringBoot & Docker + Reids 연동]]

---

# AWS의 EC2는 무엇이며, 왜 사용하는가? 에 대해 알아보겠다. 

#### EC2란 무엇인가? 
	- Amazon Elastic Compute Cloud으로써 AWS에서 제공하는 클라우드 컴퓨팅 서비스이다. 즉, 독립괸 컴퓨터를 임대해주는 aws의


오늘은 EC2에서 인스턴스를 생성하고, docker를 통해 Spring boot 프로젝트를 띄워보겠다.
천천히 따라가 보자.

---

## 1. DOCKER 셋팅

### _1.1 SpringBoot프로젝트에 Docker파일 생성하기_

_먼저  프로젝트가 설치된 경로에서 ' mvn install '명령어를 사용하여 jar 파일을 생성한다_

![[Pasted image 20250221114856.png]]
![[Pasted image 20250221114901.png]]

### _1.2 Docker파일로 image 생성_

```
docker build -t [이미지 이름]:[태그] [dockerfile이 저장된 경로]
```

![[Pasted image 20250221114932.png]]

### _1.3 생성한 image를 dockerhub에 **push**_ 

![[Pasted image 20250221114945.png]]

![[Pasted image 20250221114951.png]]

이제 docker는 준비가 끝났다...

---

## 2. EC2 셋팅

### _2.1  EC2 인스턴스 생성_

자세한 설명은 생략한다. `검색해 보시길`

![[Pasted image 20250221114959.png]]

### _2.2 쉘스크립트로 들어가기_

4.번의 주소를 사용해서 접속한다.
이때!, **.pem** 파일이 저장 되어있는 경로에서 진행한다

![[Pasted image 20250221115004.png]]

접속완료.

![[Pasted image 20250221115010.png]]

### _2.3 쉘스크립트에서 docker 다운로드 및 image PULL 받기_

먼저, aws 무료 버전 인스턴스를 생성하고자 한다.
1. aws 사이트에 접속하여 개인계정을 생성한다. (aws 프리티어 서비스를 이용하면 1년 무료로 사용가능 함)
#### 쉘스크립트에서 도커를 다운받은 후,  1번에서 올려놓은 image를 PULL 받는다.

```
sudo docker pull [image이름]:[태그]
```

![[Pasted image 20250221115019.png]]

```
sudo docker images
```

명령어를 통해 생성된것을 확인할 수 있다.

![[Pasted image 20250221115024.png]]

#### ※ 나는 Apple m1 mac을 사용 중 이기 때문에 호환성 문제로 linux환경에 맞는 이미지를 한개 더 올렸다.

1.2의 과정에서 ' --platform linux/amd64 ' 태그를 추가해 주고, 태그에 -linux를 붙였다.

![[Pasted image 20250221115030.png]]

### _2.4 image컨테이너로 실행 시키기_

![[Pasted image 20250221115035.png]]

잘 따라왔다면 EC2 환경에서 docker를 통해 받은 jar파일이 잘 실행되는 것을 볼 수 있다.

내가 만든 포트는 8000번이며 IP주소는 AWS의 인스턴스에서 찾아볼 수있다.

**퍼블릭 IP주소 를 통해 접속을 확인해보기 전에..**

![[Pasted image 20250221115040.png]]

해당 인스턴스의 보안그룹을 확인하고, 내가 설정한 포트(:8000)에 대한 접근을 허가해주어야한다.
보안그룹 확인 후,

![[Pasted image 20250221115046.png]]

네트워크 및 보안 -> 해당 보안그룹 ->  인바운드 규칙 편집 ( **인바운드**란, 외부에서 해당 인스턴스로 접근하는 것)

![[Pasted image 20250221115052.png]]

**8000번 포트 추가**

![[Pasted image 20250221115058.png]]

이렇게 과정을 마치면...
```
http://[인스턴스의 퍼블릭 IP주소]:[포트번호]
```
 로 접속이 가능하다!
![[Pasted image 20250221115102.png]]

---

# + <font color="#c0504d">Redis</font>도 함께 올려보자!

## AWS EC2에 Redis를 설치하고, Spring 프로젝트까지 연동을 해볼것이다.

다음의 대략정인 과정을 통해 진행될 것이다.

_1. Docker를 통해 Redis 다운받기_

_2. Redis config 파일 생성/수정 및 docker file 생성하기_

_3. 2에서 생성한 conf 파일과 dockerfile로 docker image 생성하기_

_4. 생성한 image 를 docker hub에 올리기_

_5. EC2에서 redis image와 spring(jar)image 내려받기_

_6. 내려받은 image를 container로 실행하고, EC2 포트 열기_

EC2셋팅법과 docker 및 redis를 셋팅하는 방법은 이전 글에서 찾아볼 수 있다.

---

_spring + docker 셋팅_
# [[📘 SpringBoot & Docker + Reids 연동]] 를 먼저 보고오면 도음이 됩니다.

---

## 1. docker 로 redis 최신으로 내려받기

로컬에 도커가 셋팅되어있다고 가정하고,  redis를 최신으로 내려받는다.

```
docker pull redis
```

![[Pasted image 20250221115117.png]]

docker desktop을 통해 확인할 수 있고, 

 ```
docker images
```
명령어를 통해서 확인 할 수 있다.

![[Pasted image 20250221115122.png]]

---

## 2.  redis.conf 파일 생성 및 docker file 생성

1.에서 redis를 내려받았다면 해당하는 디렉토리에 redis.conf 파일이 생성된다.
	 *생성되지 않았을경우 text파일 형식으로 생성해도 문제없다.*

생성된 conf파일을 다음과 같이 수정하였다.

```
# 연결 가능한 네트위크(0.0.0.0 = Anywhere)
bind 0.0.0.0

# 연결 포트
port 6379

# Master 노드의 기본 사용자 비밀번호
requirepass 사용할비밀번호입력

# 최대 사용 메모리 용량(지정하지 않으면 시스템 전체 용량)
maxmemory 2gb

# 설정된 최대 사용 메모리 용량을 초과했을때 처리 방식
# - noeviction : 쓰기 동작에 대해 error 반환 (Default)
# - volatile-lru : expire 가 설정된 key 들중에서 LRU algorithm 에 의해서 선택된 key 제거
# - allkeys-lru : 모든 key 들 중 LRU algorithm에 의해서 선택된 key 제거
# - volatile-random : expire 가 설정된 key 들 중 임의의 key 제거
# - allkeys-random : 모든 key 들 중 임의의 key 제거
# - volatile-ttl : expire time(TTL)이 가장 적게 남은 key 제거 (minor TTL)
maxmemory-policy volatile-ttl

# == RDB 관련 설정 ==
# 저장할 RDB 파일명
dbfilename backup.rdb
# 15분 안에 최소 1개 이상의 key가 변경 되었을 때
save 900 1
# 5분 안에 최소 10개 이상의 key가 변경 되었을 때
save 300 10
# 60초 안에 최소 10000개 이상의 key가 변경 되었을 때
save 60 10000
# RDB 저장 실패 시 write 명령 차단 여부
stop-writes-on-bgsave-error no

# == AOF 관련 설정 ==
# AOF 사용 여부
appendonly yes
# 저장할 AOF 파일명
appendfilename appendonly.aof
# 디스크와 동기화 처리 방식
# - always : AOF 값을 추가할 때마다 fsync를 호출해서 디스크에 쓰기
# - everysec : 매초마다 fsync를 호출해서 디스크에 쓰기
# - no : OS가 실제 sync를 할 때까지 따로 설정하지 않음
appendfsync everysec

# == Replication 관련 설정테스트 ==
# Slave Redis 설정
#임시주석slaveof 127.0.0.1 6380
```

docker file도 생성하여 준다.

```
FROM redis:latest
COPY redis.conf /저장디렉토리/redis.conf
CMD [ "redis-server", "/저장디렉토리/redis.conf" ]
EXPOSE 6379
```

![[Pasted image 20250221115129.png]]

---

## 3.  dockerfile로 원본 redis 이미지를 사용하여 EC2용 이미지 생성하기

```
docker build --platform linux/amd64 -t 이미지이름:태그 디렉토리
----
docker build --platform linux/amd64 -t springredis:linux .
```

dockerfile이 있는 디렉토리에서 해당 명령어를 실행한다.

_* --platform linux/amd64 태그는 본인의 ec2환경이 리눅스64 환경이기 때문에 추가했다._

![[Pasted image 20250221115132.png]]

---

## 4.  생성한 ec2용 이미지 docker 허브에 push하기

```
docker push xotjd794613/springredis:linux
------
docker push 계정명/이미지이름:태그
```

![[Pasted image 20250221115136.png]]

![[Pasted image 20250221115140.png]]

업로드까지 완료했다.

---

## 5.  ec2 ssh에서 도커로 이미지 pull받고 컨테이너 실행하기

이전에 만들어두었던 **Spring(jar)이미지** 와 방금 push한 **redis이미지를 받는다**

```
docker pull xotjd794613/funfun:0.0.1-linux
docker pull xotjd794613/springredis:linux
--
docker pull 계정명/이미지이름:태그
```

```
docker images
```
명령어로 확인 할 수 있다.

![[Pasted image 20250221115146.png]]

이미지를 컨테이너로 실행하자.

**Spring 이미지 실행**

```
sudo docker run -d -p 8000:8080 xotjd794613/funfun:0.0.1-linux
------
sudo docker run -d -p 포트 계정명/이미지명:태그
```

**redis 이미지 실행**

```
docker run --name springredis -p 6379:6379 -v /home/ec2-user/redis:/data -d xotjd794613/springredis:linux --appendonly yes

----------

docker run --name 컨테이너이름 -p 포트 -v 데이터저장할디렉토리:/data -d 계정명/이미지명:태그 --appendonly yes
```

'docker ps -a' 명령어로 실행중인 모든 컨테이너를 확인 할 수 있다.

![[Pasted image 20250221115151.png]]

**http://ec2주소:포트번호로 접속시 정상적으로 spring기반 페이지와 redis가 연결된 것을 확인할 수 있다.**

![[Pasted image 20250221115157.png]]

---

>[!failure] 오류
>만약, 해당주소로 접속시 접속이 안되거나, redis가 정상적으로 실행되지 않는경우

## 1. EC2의 인바운드 규칙에 redis포트번호를 추가하여 준다
-
![[Pasted image 20250221115201.png]]

![[Pasted image 20250221115214.png]]

---

## 2. Spring 프로젝트의 properties파일의 redis host를 ec2 redis 컨테이너의 ip로 잡아준다.

**2.1 ec2 redis 컨테이너 포트 확인**

```
docker ps -a
```
로 redis 컨테이너 ID 확인

![[Pasted image 20250221115220.png]]

```
docker inspect 컨테이너ID
```
명령어로 IP 확인

![[Pasted image 20250221115224.png]]

![[Pasted image 20250221115228.png]]

spring 프로젝트에 ip 정보 추가
![[Pasted image 20250221115232.png]]

---