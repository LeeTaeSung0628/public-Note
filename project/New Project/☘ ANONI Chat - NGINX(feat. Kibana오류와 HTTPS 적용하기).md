# ☘ ANONI Chat - NGINX(feat. Kibana오류와 HTTPS 적용하기)

#프로젝트 #개발 #보안 #인프라 #HTTPS #트러블슈팅 #NGINX

---

<br>

#### 지난 포스트
#### ▶ [[☘ ANONI Chat - infra setup]]
#### ▶ [[☘ ANONI Chat - CICD 구성]]
#### ▶ [[☘ ANONI Chat - ELK Stack setting]]



---

<br>

# <font color="#76923c">개요</font>

- elasticsearch 라이센스 및 보안 문제

<u>Kibana에 접속시, UI가 안보이고 오류로그가 JSON 형태로만 보이는 오류가 발생.</u>

![[do-messenger_screenshot_2025-06-09_17_27_36.png|600]]

찾아보니 *elasticsearch 7.11+* 버전에서 발생하는 문제라고 한다.

<br>

우선 임시방편으로 docker-compose.yml을 수정할 필요가 있다.  
<u>라이센스를 basic으로 명시해줘야하고 보안 설정을 false로 해야한다.</u>

보안 설정을 해제하는 이유는 보안 설정을 하면 HTTPS 사용이 강제되어서 HTTP로는 접근이 불가능하기 때문이다.  
>[!info] 정보
>현재는 환경 구축 단계이고 **HTTPS**를 적용하지 않은 상태이기 때문에 나중에 **HTTPS** 설정을 하고 보안 설정을 다시 할 예정이다.

docker-compose.yml 변경 사항

```yml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=false     # 보안 기능 비활성화
    - xpack.license.self_generated.type=basic  # 기본 라이센스 사용
  ports:
    - "9200:9200"
  networks:
    - elk
  volumes:
    - esdata:/usr/share/elasticsearch/data

kibana:
  image: docker.elastic.co/kibana/kibana:7.11.1
  ports:
    - "5601:5601"
  networks:
    - elk
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    - server.host=0.0.0.0
    - xpack.security.enabled=false     # Kibana 보안 비활성화
    - xpack.license.self_generated.type=basic
```

- 정상 출력
![[Pasted image 20250609174528.png]]
---

<br>

# <font color="#76923c">HTTPS + NGINX 적용하기</font>

<br>

## HTTPS와 NGINX는 무엇인가?

#### HTTPS란?
- HTTPS(HyperText Transfer Protocol Secure)는 HTTP에 **SSL/TLS 암호화를 추가한** 보안 <u>통신 프로토콜</u>이다.
1. *암호화* : 데이터가 전송 중에 도청당하지 않도록 암호화
2. *무결성* : 데이터가 중간에 조작되지 않았는지 검증
3. *인증* : 서버의 신원을 보장(브라우저의 자물쇠 표시)

- SSL인증서
- <u>포트 : 443번 (https://URL에 포트명 생략되어있음, http://URL 은 80포트)</u>

<br>

#### NGINX란?
- 엔진엑스는 웹서버이자 **리버스 프록시**서버로, 정적 파일을 제공하고, 트래픽 분산, **HTTPS 처리**, 도메인 라우팅 등 다목적 기능을 수행하는 <u>경량 고성능 서버</u>이다.
1. *정적 웹 서버* : HTML, CSS, JS 등 정적 리소스를 클라이언트에 서빙
2. *리버스 프록시* : 클라이언트 요청을 백엔드(Spring 등)에 전달
3. *로드 벌런싱* : 여러 백엔드 서버에 트래픽 분산
4. *HTTPS 종단 처리* : SSL인증서를 이용해 HTTPS 연결처리

<br>

## HTTPS와 NGINX의 관계도
```python
[ 클라이언트 (브라우저) ]
        ⇩ HTTPS 요청 (443)
┌──────────────────────┐
│      NGINX 서버       │
│ - SSL 인증서 보유      │
│ - HTTPS 처리          │
│ - 요청을 Spring에 전달 │
└─────────┬────────────┘
          ⇩ HTTP (80)
    [ Spring Boot 서버 ]
```

- **SSL 오프로드** : SSL 암호화 부담을 NGINX가 맡아 백엔드는 단순 HTTP로 처리
- **인증서 관리 용이** : 여러 백엔드에 인증서 분산하지 않고 NGINX 한 곳에서 관리
- **도메인 기반 라우팅 가능** : 여러 도메인 요청을 각기 다른 백엔드로 라우팅 가능 (`/api`, `/admin` 등)

>[!note] 정리
> HTTPS등 인증서를 Spring에서 직접처리해도 되지만,
> NGINX에서 프록시 + 인증서 처리를 담당하는 것이 확장성/유지보수 측면에서 유리하다.


---

<br>

## <u>NGINX + certbot</u> / <u>NginxProxyManager(NPM)</u>

>[!tip] certbot이란?
>**SSL 인증서 발급 자동화 도구**
>- 만든 곳: [EFF (Electronic Frontier Foundation)](https://eff.org/)
>- 목적: **Let's Encrypt**의 무료 인증서를 사용자 시스템에 자동으로 설치하고, NGINX/Apache 설정까지 자동으로 수정함
>- 인증서 형식: **도메인 기반 DV 인증서**

<br>
*Nginx+certbot*, *NPM* 두개는 어떠한 차이가 있는가?

#### 1. NGINX + certbot
- 전통적인 수동/스크립트 기반 HTTPS 구성
- 리버스 프록시, 정적 파일 서빙, 로드밸런싱 등을 자유롭게 구성 가능

|구성 요소|설명|
|---|---|
|**NGINX**|리버스 프록시, 웹 서버|
|**certbot**|[Let's Encrypt](https://letsencrypt.org/) 무료 인증서 발급 클라이언트|
|**.conf 파일**|직접 작성해야 함 (예: `server { listen 443; ... }`)|
|**cron/script**|인증서 자동 갱신 후 NGINX 재로드 수동 설정 필요|
장점
- 유연하며, 다양한 프록시를 구성할수 있다.
-  docker와 조합해도 설정이 자유롭다.

단점
- 설정이 복잡하다.
- 인증서를 수동갱신 해주어야하며, 자동화시 추가 설정이 필요하다.
- 실수 시 서비스 중단 가능성이 있다.

<br>

#### 2. NGINX Proxy Manager (NPM)
- 운영 자동화를 위한 GUI 기반 NGINX 관리 툴
- Docker 기반 설치 + 웹 UI 제공

| 구성 요소                        | 설명                                  |
| ---------------------------- | ----------------------------------- |
| **Nginx Proxy Manager 컨테이너** | 내부적으로 NGINX + certbot + 관리 인터페이스 내장 |
| **Web UI**                   | 웹 브라우저로 리버스 프록시/도메인 설정/HTTPS 적용 가능  |
| **자동 인증서 발급**                | Let's Encrypt 인증서 자동 신청/갱신 내장       |
| **Docker 기반 구성**             | 한 줄로 실행 가능 `docker-compose` 스택 제공   |
장점
- GUI환경에서 리버스 프록시와 SSL설정을 간편하게 가능하다.
- 인증서 갱신의 자동처리
- 사용로그 및 실패이력 자동관리

단점
- 고급설정에 제한적이다.
- 커스텀 동작을 설정하는데에도 제한 있음.
- 로드벨런싱 등 역할 수행이 제한적이다.

→ 로드 벨런싱등 고급 설정은 추후 **k8s**를 통해 구현될 예정이기 때문에 상관없다.


#### 위와 같은 상황을 고려했을 때, **NPM**을 선택.

---

<br>

## NPM 셋팅하기

- NPM 셋팅 이전, DB셋팅 먼저 진행하도록 하였다.

## ▶ [[ANONI Chat - DB setup]]


이후, NPN을 다운로드 받는다.

# 

HTTPS 적용

Nginx + certbot으로 https를 적용하는 과정이다.

### 

Nginx의 역할

HTTPS 적용의 입장에서만 보면 nginx는 특정 도메인으로 들어오는 모든 요청을 HTTPS로 리다이렉트 해주는 역할을 해주고  
리버스 프록시로서 HTTPS를 HTTP로 변경해서 서버로 전달해준다(443 > 80)  
또한 ssl 인증서를 관리하는 역할을 수행한다.

### 

certbot의 역할

- **인증서 발급** - Let's Encrypt CA에서 무료 SSL 인증서 받아옴
- **인증서 갱신** - 만료 전 자동으로 새 인증서 발급
- **파일 저장** - 인증서를 특정 디렉토리에 저장
- **nginx 알림** - 갱신 후 nginx에게 새 인증서 사용하라고 신호

certbot은 인증서를 발급/갱신하고 특정 디렉토리에 저장  
그리고 nginx에게 알림을 주는 역할을 한다.

## 

적용 순서

### 

1. docker-compose.yml 수정

기존 도커 컴포즈 파일

```yml
version: '3'

  

services:

  elasticsearch:

    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1

    environment:

      - discovery.type=single-node

      - xpack.security.enabled=false

      - xpack.license.self_generated.type=basic

    ports:

      - "9200:9200"

    networks:

      - elk

    volumes:

      - esdata:/usr/share/elasticsearch/data

  

  logstash:

    image: docker.elastic.co/logstash/logstash:7.12.0

    ports:

      - "5044:5044"

      - "5000:5000"

    volumes:

      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

    networks:

      - elk

    depends_on:

      - elasticsearch

  

  kibana:

    image: docker.elastic.co/kibana/kibana:7.11.1

    ports:

      - "5601:5601"

    networks:

      - elk

    environment:

      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

      - server.host=0.0.0.0

      - xpack.security.enabled=false

      - xpack.license.self_generated.type=basic

    depends_on:

      - elasticsearch

  

  spring:

    image: ghcr.io/anonichat/app/anonichat

    ports:

      - "80:8080"

    environment:

      - ELASTICSEARCH_HOST=elasticsearch:9200

      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/anonichat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul

      - SPRING_DATASOURCE_USERNAME=anonichat

      - SPRING_DATASOURCE_PASSWORD=비번

    depends_on:

      - elasticsearch

      - mysql

    networks:

      - elk

      - data

  

  mysql:

    image: mysql:8.0

    container_name: mysql

    restart: unless-stopped

    ports:

      - "3306:3306"

    environment:

      - MYSQL_DATABASE=anonichat

      - MYSQL_ROOT_PASSWORD=비번
      - MYSQL_USER=anonichat

      - MYSQL_PASSWORD=비번

    volumes:

      - mysql_data:/home/hello/Desktop/AnoniChat/elk-stack/data/mysql

    networks:

      - elk

      - data

networks:

  elk:

  data:

  

volumes:

  esdata:

  mysql_data:
```

기존 도커 컴포즈 파일에 nginx와 certbot을 추가한다.

**수정한 도커 컴포즈 파일**

```yml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - xpack.license.self_generated.type=basic
    ports:
      - "9200:9200"
    networks:
      - elk
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.12.0
    ports:
      - "5044:5044"
      - "5000:5000"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.11.1
    ports:
      - "5601:5601"
    networks:
      - elk
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - server.host=0.0.0.0
      - xpack.security.enabled=false
      - xpack.license.self_generated.type=basic
    depends_on:
      - elasticsearch

  # Spring Boot 앱 (포트 변경 중요!)
  spring:
    image: ghcr.io/anonichat/app/anonichat
    # 외부 포트 제거 - nginx를 통해서만 접근
    expose:
      - "8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/anonichat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul
      - SPRING_DATASOURCE_USERNAME=anonichat
      - SPRING_DATASOURCE_PASSWORD=비번
    depends_on:
      - elasticsearch
      - mysql
    networks:
      - elk
      - data

  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=anonichat
      - MYSQL_ROOT_PASSWORD=비번
      - MYSQL_USER=anonichat
      - MYSQL_PASSWORD=비번
    volumes:
      - mysql_data:/home/hello/Desktop/AnoniChat/elk-stack/data/mysql
    networks:
      - elk
      - data

  # Nginx 리버스 프록시
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    depends_on:
      - spring
    networks:
      - elk  # Spring과 같은 네트워크에 연결
    command: ["nginx", "-g", "daemon off;"]

  # Certbot SSL 인증서 관리
  certbot:
    image: certbot/certbot
    container_name: certbot
    restart: unless-stopped
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
      - /var/run/docker.sock:/var/run/docker.sock
    entrypoint: "/bin/sh"
    command: > # ssl 인증서 갱신시에만 nginx 설정 reload하는 명령어
      -c "
      apk add --no-cache docker-cli;
      while :; do
        echo 'Checking for certificate renewal...';
        certbot renew --deploy-hook 'docker exec nginx nginx -s reload';
        sleep 12h;
      done
      "

networks:
  elk:
  data:

volumes:
  esdata:
  mysql_data:
```

**spring의 port 변경**

```yml
# 기존 설정
ports: - "80:8080" # 호스트포트:컨테이너포트

# 변경된 설정
expose: - "8080" # 컨테이너포트만

# nginx의 설정
ports:
      - "80:80"
      - "443:443"
```

기존 설정은 외부를 통해서 컨테이너에 접근이 가능했다.  
브라우저 → 서버:80 → Spring Boot:8080  
이렇게 외부에서 80 포트로 요청이 들어오면 호스트(인스턴스)의 80포트를 통해 스프링 컨테이너의 8080포트로 전달해주는 방식

변경된 설정은 컨테이너의 8080만 열어놓는다.  
이렇게 되면 호스트의 80포트로 들어오는 요청은 무조건 nginx를 거쳐서 들어오게 된다.  
브라우저 → 서버:80 → nginx:80 → spring:8080

HTTP 요청이 프록시를 거쳐서 오게끔 구성한 설정이다.

### 

2. 디렉토리 생성

```
# docker-compose.yml이 있는 폴더에서 실행
mkdir nginx
# nginx 하위 폴더 생성
mkdir conf.d

# docker-compose.yml이 있는 폴더에서 실행
mkdir certbot
#certbot 하위 폴더 생성
mkdir conf
mkdir www
```

### 

3. Nginx 설정 파일 생성 및 ssl 인증서 발급

**인증서 발급 전 임시 설정**

```
upstream spring-backend {
    server spring:8080;
}

# HTTP 서버 (인증서 발급 전 임시 설정)
server {
    listen 80;
    server_name anonichat.world www.anonichat.world;
    server_tokens off;

    # Let's Encrypt 도메인 인증용 경로
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # AnoniChat 애플리케이션 (HTTP로 임시 서비스)
    location / {
        proxy_pass http://spring-backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        # WebSocket 지원 (채팅 기능용)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

**도커 컴포즈 재시작**

```
docker-compose up -d
```

**ssl 인증서 발급**

```
docker run --rm -it \                                                                                                                         --name certbot \                                               

  -v $(pwd)/certbot/conf:/etc/letsencrypt \

  -v $(pwd)/certbot/www:/var/www/certbot \

  certbot/certbot \

  certonly --webroot \

  --webroot-path /var/www/certbot \

  --email jsi50069@gmail.com \

  --agree-tos \

  --no-eff-email \

  -d anonichat.world \

  -d www.anonichat.world
```

certbot으로 ssl 인증서를 받음

**인증서 확인**

```
docker-compose exec certbot ls -la /etc/letsencrypt/live/anonichat.world/
```

**certbot 재시작**

```
docker-compose up -d certbot
```

**nginx 설정 파일 변경**

```
upstream spring-backend {

    server spring:8080;

}

  

# HTTP → HTTPS 강제 리다이렉트

server {

    listen 80;

    server_name anonichat.world www.anonichat.world;

    server_tokens off;

  

    # Let's Encrypt 경로만 HTTP 허용

    location /.well-known/acme-challenge/ {

        root /var/www/certbot;

    }

  

    # 나머지 모든 요청은 HTTPS로 강제 리다이렉트

    location / {

        return 301 https://$host$request_uri;

    }

}

  

# HTTPS 서버

server {

    listen 443 ssl;

    http2 on;

    server_name anonichat.world www.anonichat.world;

    server_tokens off;

  

    ssl_certificate /etc/letsencrypt/live/anonichat.world/fullchain.pem;

    ssl_certificate_key /etc/letsencrypt/live/anonichat.world/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;

    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;

    ssl_prefer_server_ciphers off;

    ssl_session_cache shared:SSL:10m;

    ssl_session_timeout 10m;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    add_header X-Frame-Options DENY always;

    add_header X-Content-Type-Options nosniff always;

  

    location / {

        proxy_pass http://spring-backend;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header X-Forwarded-Host $server_name;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;

        proxy_set_header Connection "upgrade";

    }

}
```

### 

4. 재배포 및 테스트

**nginx 설정 파일 테스트**

```
docker-compose exec nginx nginx -t

# 명령어 결과
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**nginx 재시작**

```
docker-compose exec nginx nginx -s reload
```

**테스트**

```
curl -I http://anonichat.world
# 명령어 결과
HTTP/1.1 301 Moved Permanently

**Server**: nginx

**Date**: Thu, 12 Jun 2025 16:24:54 GMT

**Content-Type**: text/html

**Content-Length**: 162

**Connection**: keep-alive

**Location**: https://anonichat.world/
```

Location을 보면 https로 전환되는것을 볼 수 있다.

![Pasted image 20250613013521.png](https://kjsdevblog.netlify.app/image/pasted-image-20250613013521.png)  
브라우저에서도 HTTPS로 변경된 것을 확인