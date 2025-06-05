# 🤑 ANONI Chat 개발기 - ELK Stack setting

#프로젝트 #개발 #인프라 #Elasticsearch #Logstash #Kibana

---

<br>

## 이전 셋팅 보러가기
#### ▶ [[🤑 ANONI Chat 개발기 - infra setup]]
#### ▶ [[🤑 ANONI Chat 개발기 - CICD 구성]]

---

<br>

# <font color="#76923c">ELK란 무엇인가?</font>

- ELK는 **Elasticsearch, Logstash, Kibana**의 약자로, **로그 수집, 저장, 분석, 시각화**를 위한 오픈소스 로그 플랫폼 스택이다.
- 최근에는 Beats까지 포함한 **"Elastic Stack"** 이라고도 부른다.

![[do-messenger_screenshot_2025-06-04_13_38_39.png]]

## ELK Stack의 구성 요소

|구성 요소|역할 요약|상세 설명|
|---|---|---|
|**Elasticsearch**|로그 저장 & 검색 엔진|고속 검색 가능한 NoSQL 기반의 분산형 문서 데이터베이스|
|**Logstash**|로그 수집 & 가공 도구|다양한 형식의 로그를 수집하고, 필터링 및 변환 후 Elasticsearch로 전달|
|**Kibana**|로그 시각화 UI|Elasticsearch의 데이터를 시각화해주는 웹 인터페이스|
|**(Beats)**|로그 전달 경량 에이전트|Filebeat, Metricbeat 등. 각 시스템에서 데이터를 Logstash/ES로 보냄|

<br>

#### ELK 장점

| 상황                                        | ELK 적합 여부    |
| ----------------------------------------- | ------------ |
| 서버가 많아 *로그가 분산*되어 있는 경우                   | ✅ 중앙집중화      |
| 에러, 경고, 사용자 행위를 빠르게 추적하고 싶은 경우            | ✅ 실시간 검색     |
| 로그에서 통계 분석 및 대시보드를 구성하고 싶은 경우             | ✅ 시각화        |
| JSON 형식의 로그를 수집 및 필터링하고 싶은 경우             | ✅ 구조화된 로그 처리 |
| DevOps, SRE, 보안팀이 *로그 기반 모니터링/알림*이 필요한 경우 | ✅ 필수 도구      |

#### ELK 단점
- 고성능을 위해 많은 메모리가 필요하다.
- infra를 셋팅하는데 있어서 **러닝커브가 높다**..

---

<br>

# <font color="#76923c">SpringBoot 프로젝트 세팅</font>

<br>

## Spring dependencies 추가


*build.gradle*
```java
dependencies {  
    implementation 'net.logstash.logback:logstash-logback-encoder:7.4'  
}
```
- logstash 의존성을 추가해준다.

<br>

## logback-spring.xml 설정 추가
- spring boot용 Logback 로깅 사용자 정의 설정파일

<br>

*logback-spring.xml*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration> <!--모든 로그를 콘솔에 출력-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder> <!--예: `12:30:15.321 [main] INFO AuctionService - Started`-->
    </appender>

    <appender name="MAIN_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">

        <destination>logstash:5000</destination> <!--컨테이너 포트 5001으로 전송-->

        <encoder class="net.logstash.logback.encoder.LogstashEncoder" /> <!--JSON 형식 로그로 인코딩-->
        <keepAliveDuration>5 minutes</keepAliveDuration> <!--TCP연결 5분간 유지-->
    </appender>

<!-- 추가적으로 로그 분기 가능 (ex) 
		java.
		Logger logger = LoggerFactory.getLogger("AuctionServiceLogger");
        logger.info("{}", bidLogDTO);
        
    <appender name="CUSTOM_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">

        <destination>logstash:5001</destination>

        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
        <keepAliveDuration>5 minutes</keepAliveDuration>
    </appender>
-->

	<!--전체 시스템 로그 중 INFO 이상만 콘솔 출력 (별도 logger 설정 없는 경우에 해당)-->
    <root level="info">
        <appender-ref ref="CONSOLE" />
    </root>

	<!--
	클래스 또는 패키지 이름이 `AuctionServiceLogger`인 로거에 적용
	DEBUG 이상 로그
    additivity="false" : 루트로 로그 전파 X (CONSOLE + AUCTION_LOGSTASH만 사용)
    + 콘솔 동시 출력
	-->
    <logger name="MainServiceLogger" level="debug" additivity="false">

        <appender-ref ref="AUCTION_LOGSTASH" />
        <appender-ref ref="CONSOLE" />
    </logger>

<!--
    <logger name="CustomServiceLogger" level="debug" additivity="false">
        <appender-ref ref="CUSTOM_LOGSTASH" />
        <appender-ref ref="CONSOLE" />
    </logger>
-->
</configuration>
```

<br>

## TEST용 log 생성

*mainController*
```java
@GetMapping(GlobalURL.MAIN_URL)  
public ModelAndView mainView()  
{  
    log.info("[MainController Log] mainView 접속 TEST");  
    return new ModelAndView("main");  
}
```

---

<br>

# <font color="#76923c">Ubuntu server Kibana 사용 셋팅</font>

<br>

## docker-compose.yml 생성
- DockerComposeTool 설정파일.
- <u>여러개의 컨테이너(서비스)를 하나의 애플리케이션 처럼 정의하고 실행하도록 도움.</u>
- 컨테이너 환경을 코드화/자동화
- 컨테이너를 띄울 서버(필자는 Ubuntu)에 생성하여 준다.
<br>

1. Elasticsearch
2. Logstash
3. Kibana
4. Spring Boot 애플리케이션

주석 제외코드 ▶ [[anoniChat-docker-compose.yml]]
```yml
# Docker Compose 파일 스펙 버전 3 사용
version: '3' 

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
    environment:
      - discovery.type=single-node # 단일 노드 구성
    ports:
      - "9200:9200"
    networks:
      - elk # `elk`키워드 네트워크로 구성 (다른 서비스와 내부 통신)
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
	  image: docker.elastic.co/logstash/logstash:7.12.0
	  ports:
	    - "5044:5044" # Filebeat 등 input으로 사용하는 포트
	    - "5000:5000" # TCP 또는 JSON 로그 input 용 포트 (Spring에서 이 포트를 사용)
	  volumes:
	    - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
	  networks:
	    - elk # `elk`키워드 네트워크

  kibana:
    image: docker.elastic.co/kibana/kibana:7.11.1
    ports:
      - "5601:5601" # 웹 UI 접근용 포트
    networks:
      - elk # `elk`키워드 네트워크
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200 # Elasticsearch 주소 연결
      - server.host=0.0.0.0 # 모든 IP 바인딩 허용

  spring:
    image: ghcr.io/anonichat/app/anonichat
    ports:
      - "8080:8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200 # Elasticsearch의 내부 주소를 환경변수로 주입
    depends_on:
      - elasticsearch # Elasticsearch가 먼저 실행되도록 보장
    networks:
      - elk # 내부 ELK 네트워크로 연결

volumes:
  esdata:
    driver: local # Elasticsearch 데이터 저장소를 호스트 볼륨에 영구 저장

networks:
  elk: # 모든 서비스가 하나의 공용 네트워크 `elk`에서 통신
    driver: bridge # `elasticsearch`, `logstash`, `spring`, `kibana`는 서로 이름으로 접근 가능
```


<br>

# logstash.conf 생성
- <u>Logstash의 데이터 처리 파이프라인을 정의</u>하는 설정 파일이다.
- `docker-compose.yml`파일을 생성한 같은 디렉토리에 생성한다.


주석 제외 코드 ▶ [[anoniChat-logstash.conf]]
```c
input { // 데이터 수신 설정

	beats {
	    port => 5044
	}
	  
	tcp {
	    port => 5000
	    codec => json_lines // json형식으로 한줄씩 파싱
	    type => "main_log" // 수산 로그에 type필드로 "auction_log" 부여
	}
	
	//tcp {
	//    port => 5001
	//    codec => json_lines
	//    type => "custom_log"
	//}
}

filter { // 수신된 로그를 처리하기위한 전처리
	if [type] == "main_log" { // 로그 타입이 `main_log`일 때만 처리.
		grok { // 정규식으로 메세지 파싱
			match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:loglevel} %{DATA:logger} - %{GREEDYDATA:logmessage}" }
		}
	}

// 필터 사용 예시
//	if [type] == "custom_log" {
//		if "TestLogDTO" in [message] {
//		    grok {
//		        match => {
//		           "message" => "TestLogDTO\(userId=%{NUMBER:user_id}, exchangeAmount=%{NUMBER:exchange_amount}, payType=%{WORD:pay_type}, payStatus=%{WORD:pay_status}\)"
//		        }
//		    }
//		    mutate { // 필드 타입 변환 및 메시지 필드 제거
//			    remove_field => ["message"]  
//			}
//		} else {
//			drop { } // `TestLogDTO`가 포함되지 않으면 해당 로그 삭제(drop).
//		}
//	}
}

output { // 로그 출력 설정 시작
	if [type] == "auction_log" {
	    elasticsearch {
	      hosts => ["http://elasticsearch:9200"] // Elasticsearch로 전송
	      index => "main_log" // 인덱스 이름: `main_log`.
		}
	}
//  if [type] == "custom_log" {
//    elasticsearch {
//      hosts => ["http://elasticsearch:9200"]
//      index => "custom_log"
//   }
//  }
}
```

<br>

---

<br>

# <font color="#76923c">적용 및 확인</font>


<br>

## server에 파일 생성하기

<br>

임의의 폴더를 지정한 후,

[[anoniChat-docker-compose.yml]] / [[anoniChat-logstash.conf]]

```bash
nano anoniChat-docker-compose.yml
nano logstash.conf
```
파일을 생성한다.

![[Pasted image 20250605172214.png|550]]

>[!info] DockerComposeTool이 없다면?
>```bash
> sudo apt install -y docker-compose
>```
>해당 명령어로 다운로드 받기


이후,
```bash
docker-compose up -d
```
- `-d` : 백그라운드로 실행

`http://{‘IP주소‘}:5610 (Kibana port)` 로 접속확인

![[Pasted image 20250605172823.png]]

<br>

Kibana를 사용할 준비가 되었다.

이후, 왼쪽의 메뉴바 에서 `Analytics` → `Discover` 메뉴로 이동
