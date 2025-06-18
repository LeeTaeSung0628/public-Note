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

>[!info] ‘리버스 프록시’라고 불리는 이유?
>**정방향 프록시**는:  
 >   - 사용자가 **인터넷 상의 여러 서버에 접근**할 때 → 사용자의 프라이버시 보호 목적
 >   - 사용자 입장에서 중간에 "프록시 서버"가 있음.   
>
>**리버스 프록시**는:
>- **하나의 공개된 서버(예: NGINX)**가 외부 요청을 받아 내부 여러 서버로 **"역으로" 분기**시켜줌.       
>- 사용자는 실제 백엔드 서버의 존재를 **모름**.

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
>- **인증서 발급** - Let's Encrypt CA에서 무료 SSL 인증서 받아옴
>- **인증서 갱신** - 만료 전 자동으로 새 인증서 발급
>- **파일 저장** - 인증서를 특정 디렉토리에 저장
>- **nginx 알림** - 갱신 후 nginx에게 새 인증서 사용하라고 신호

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
- 조금더 무겁다.

→ 로드 벨런싱등 고급 설정은 추후 **k8s**를 통해 구현될 예정


#### 위와 같은 상황을 고려했을 때, 현재는 경량화를 위해 **Nginx + certbot**을 선택.

---

<br>

# <font color="#76923c">Nginx + certbot 셋팅하기</font>

- NPM 셋팅 이전, <u>DB셋팅</u> 먼저 진행하도록 하였다.

## ▶ [[ANONI Chat - DB setup]]

<br>

## 1. docker-compose.yml 수정

<br>

기존 docker-compose.yml 파일에 *nginx*와 *certbot*을 추가한다.

```yml
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
```

<br>

동일한 docker-compose.yml 파일 내의 *spring의 port 변경*

#### 기존 설정
```yml
Spring:
	# Spring의 기존 포트 설정
	ports: - "80:8080" # 호스트포트:컨테이너포트
```
- Host의 *80포트*를 컨테이너의 *8080포트*에 연결.
- 외부 브라우저(클라이언트)가 호스트의 *IP:8080*으로 요청보낼 수 있음.
- 즉, *컨테이너의 내부포트(8080)* 를 외부에 공개(공유).

<br><br>
#### 변경된 설정
```yml
Spring:
# Spring의 변경된 포트 설정
	expose: - "8080" # 컨테이너포트만
```
- 컨테이너 내부적으로 *8080포트*를 열어두지만, 외부에 직접 노출 X
- 다른 컨테이너가 **Docker내부 네트워크를 통해 접근만 가능.**
- 브라우저나 외부 요청을 접속 X

즉, `ports`는 외부 → 컨테이너 접근을 위해 사용하고, `expose`는 컨테이너 간 통신을 위해 사용

<br><br>

```yml
nginx:
	# + nginx의 포트 설정
	ports:
	      - "80:80"
	      - "443:443"
```
- 이후, nginx에서 리버스 프록시를 통하여,
- *브라우저* → *서버(Host):80* → *nginx컨테이너:80* → *spring컨테이너:8080*
- 의 형태로 흐름이 이어지게 된다.

<br>


## 2. Nginx 설정 파일 생성 및 ssl 인증서 발급

<br>

#### ssl 인증서 발급

```bash
docker run --rm -it \
  --name certbot \
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
- `--rm`: 컨테이너 종료 시 자동 삭제 (임시용 실행)
- `-it`: interactive 모드
- `-v ...:/etc/letsencrypt`: 인증서 저장 디렉토리 (호스트에 영구 저장)
- `-v ...:/var/www/certbot`: 인증 도메인 소유 검증에 사용하는 웹 루트 경로
- `certonly --webroot`: 웹 루트 방식으로 인증서만 발급 (nginx가 .well-known 요청을 받아줘야 함)
- `--email`: 만료 알림용 이메일
- `--agree-tos`: 이용약관 동의
- `-d`: 도메인 이름으로 ssl 인증서를 받음

<br>

#### 인증서 확인

```bash
docker-compose exec certbot ls -la /etc/letsencrypt/live/anonichat.world/
```
- 실제 인증서와 키가 저장된 경로를 확인

<br>

#### certbot 재시작

```bash
docker-compose up -d certbot
```

<br>

#### app.conf(nginx설정 파일)
- `./nginx/conf.d:/etc/nginx/conf.d` docker-compose의 해당되는 디렉토리에 생성
- <u>리버스 프록시 설정</u>

```python
upstream spring-backend {
	#Nginx가 내부 네트워크에서 Spring 컨테이너의 8080포트로 요청을 전달
    server spring:8080;
}

  

# HTTP 요청은 기본적으로 **HTTPS로 리다이렉트**
# 단, `/well-known/acme-challenge/`는 Certbot 검증을 위해 예외 허용 (정적 루트 연결)
# 이때 certbot이 바인딩한 경로와 nginx `root` 경로가 정확히 일치해야 인증 가능
server {
    listen 80;
    server_name anonichat.world www.anonichat.world;
    
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

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
    ssl_certificate /etc/letsencrypt/live/anonichat.world/fullchain.pem; #인증서 체인 파일
    ssl_certificate_key /etc/letsencrypt/live/anonichat.world/privkey.pem; # 개인 키 파일
    ssl_protocols TLSv1.2 TLSv1.3; # 지원한 TLS 버전
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384; # 강력한 암호화 스위트만 허용
    ssl_prefer_server_ciphers off; # 재사용 가능한 세션, 캐시 설정
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always; # HTTPS 강제
    add_header X-Frame-Options DENY always; # XSS 및 MIME 스니핑 방지
    add_header X-Content-Type-Options nosniff always;

	# proxy_pass 설정
    location / {
        proxy_pass http://spring-backend; # 내부 컨테이너로 요청 포워딩

		# 클라이언트의 정보(IP, 프로토콜 등) Spring서버에 전달
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade"; # WebSocket 같은 프로토콜 호환성 확보
    }
}
```

<br>

#### nginx 설정 파일 테스트

```
docker-compose exec nginx nginx -t

# 명령어 결과
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```


#### nginx 재시작

```
docker-compose exec nginx nginx -s reload
```


---

<br>

## 테스트

```bash
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

![[do-messenger_screenshot_2025-06-18_17_44_18.png]]
- Location 및 브라우저에서 **HTTPS**로 변경된 것을 확인

<br>

---

<br>

# 다음 포스트

## ▶ [[☘ ANONI Chat - 모니터링 도구 적용(Elastic APM)]]