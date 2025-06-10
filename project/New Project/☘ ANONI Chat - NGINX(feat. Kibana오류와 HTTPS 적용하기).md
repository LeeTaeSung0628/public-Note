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

- 추가예정.

