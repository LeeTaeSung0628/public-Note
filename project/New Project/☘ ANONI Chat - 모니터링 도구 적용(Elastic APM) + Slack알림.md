# ☘ ANONI Chat - 모니터링 도구 적용(Elastic APM) + Slack알림

#프로젝트 #개발 #보안 #인프라 #HTTPS #트러블슈팅 #Elasticsearch #APM #모니터링 #slack 


---

<br>

#### 지난 포스트
#### ▶ [[☘ ANONI Chat - infra setup]]
#### ▶ [[☘ ANONI Chat - CICD 구성]]
#### ▶ [[☘ ANONI Chat - ELK Stack setting]]
#### ▶ [[☘ ANONI Chat - NGINX(feat. Kibana오류와 HTTPS 적용하기)]]


---

<br>

# <font color="#76923c">개요</font>

- 기존의 셋팅된 elK-stack에 elastic APM을 적용해 보도록하겠다.

<br>

---
<br>

# <font color="#8db3e2">Elastic APM이 무엇이며, 왜 적용하는가?</font>

<br>

## APM이란?
- APM은 **A**pplication **p**erformance **M**onitoring 의 약자이다.
- 애플리케이션의 **성능정보** 및 **발생한 로그정보**, 동작중인 **서버의 Metric정보**를 수집한다.
- MSA환경에서 서비스를 구성하는 여러 앱간의 Request를 하나의 Trace로 묶어서 추적할 수 있다.(**분산 Tracing**)
- Application 지연이 발생했을 때, 지연에 대한 병목 구간을 찾아낼 수있는 모니터링 서비스이다.

>[!info] Metric 정보란?
> 시간이 지남에 따라 변화하는 데이터를 의미한다.
> 메모리 사용률, CPU 사용률, 스레드 사용률 등 **시간에 따른 추이를 추적할 가치가 있는 데이터**이다.
> 
> 어떤 서비스, 앱이냐에 따라 해석이 달라질 수는 있지만, 보편적으로 대시보드를 볼때 특정 수치들을 그래프로 보여주는 일종의 **시각화**로 이해하기도 한다.


<br><br>

## APM의 특징

- ELK의 일부로, 애플리케이션의 <u>성능을 모니터링하고, 문제의 원인을 빠르게 파악할 수 있도록</u> 설계된 도구.
- 단순한 로그 수집을 넘어서, 분산 트레이싱, 서비스 간 호출 시간 분석 등, 애플리케이션 수준의 관찰 수준을 제공

| 기능 구분                              | 설명                                                                                              |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- |
| **분산 추적 (Distributed Tracing)**    | *<u>마이크로서비스 환경</u>*에서 서비스 간 호출을 추적. 전체 요청이 여러 서비스에 걸쳐 어떻게 전파되는지 시각화.                            |
| **자동 계측 (Auto Instrumentation)**   | 많은 언어/프레임워크에서 자동으로 HTTP, DB 쿼리, 외부 호출 등의 성능 데이터를 수집.                                            |
| **에러 추적 (Error Capturing)**        | 예외 발생 시 스택 *트레이스*, 메시지, 환경정보 등을 자동 수집. 오류의 빈도, 위치, 영향을 받은 사용자 정보까지 확인 가능.                       |
| **서비스 맵 (Service Map)**            | 서비스 간의 호출 구조를 시각적으로 표현. 병목 구간이나 호출 관계를 쉽게 파악 가능.                                                |
| **실시간 메트릭 수집 (Real-Time Metrics)** | CPU 사용량, 메모리 사용량, GC 활동 등 JVM/Node.js 등의 *런타임 수준의 성능지표* 수집 가능.                                  |
| **Kibana 대시보드 통합**                 | Kibana UI를 통해 수집된 APM 데이터를 시각화. 사용자 정의 대시보드도 쉽게 생성 가능.                                          |
| **지원 언어 및 프레임워크**                  | Java, Node.js, Python, Ruby, .NET, Go 등 다양한 언어 지원. Spring Boot, Django, Express 등 주요 프레임워크도 대응. |
| **OpenTelemetry 호환**               | OpenTelemetry로 수집한 trace 데이터도 수용 가능, 벤더 종속성 낮춤.                                                 |
| **환경 정보 수집**                       | 요청 URL, 사용자 에이전트, 사용자 IP, 컨텍스트 등 다양한 환경 정보 자동 수집.                                               |
<br>

## 🆚 타 APM 도구들과 비교

| 항목      | Elastic APM         | Datadog | New Relic | Prometheus + Jaeger |
| ------- | ------------------- | ------- | --------- | ------------------- |
| 가격      | 오픈소스 기반 (자체 호스팅 가능) | 유료      | 유료        | 오픈소스                |
| 통합성     | ELK와 완벽 연동          | 자체 플랫폼  | 자체 플랫폼    | 연동 필요               |
| 시각화     | Kibana              | 자체 UI   | 자체 UI     | Grafana 필요          |
| 자동계측 범위 | 주요 프레임워크 지원         | 더 광범위함  | 매우 광범위    | 제한적                 |
| 커스터마이징  | 매우 유연               | 제한적     | 제한적       | 유연                  |

<br>

#### elastic APM은 무겁고, ELK를 모두 직접 운영해야 한다는 단점이 있지만, 그만큼 **확장성과 유연성**에 장점이 있다.

<br><br>

## Elastic APM의 구조

```text
[클라이언트 요청]
     ↓
[NGINX Reverse Proxy]
     ↓
[Spring Boot 컨테이너]
     ├── Agent가 DispatcherServlet, Service, Repository 등 Hook
     ├── Transaction 및 Span 객체 생성
     └── 비동기로 APM Server로 전송
         ↓
[APM Server]
     └── JSON 수신 후 Elasticsearch로 전송
         ↓
[Elasticsearch]
     └── 문서화된 데이터 저장
         ↓
[Kibana]
     └── 트랜잭션 시각화 및 분석

```

<br>

#### APM의 구성 요소는 크게 두 가지로 나뉜다.

<br>


## 1. APM agent - <u>애플리케이션내의 관찰자</u>

**APM Agent**는 애플리케이션 내부에 직접 탑재되어 실행 중인 코드를 **자동 또는 수동 계측**해 성능 데이터를 수집하는 라이브러리 또는 패키지다.  

Java에선 `Elastic APM Java Agent`라는 `JVM 에이전트`를 등록하여 사용.

- 메소드 실행 시간 측정
- HTTP 요청/응답 모니터링
- DB 쿼리 실행 시간 추적
- JVM 메모리, CPU 사용량 수집
- 에러/예외 발생 시 스택 트레이스 캡처
- 수집한 데이터를 *APM Server로 전송*

<br>

## 2. APM server - 데이터 수집 게이트웨이

**APM Server**는 다양한 APM Agent가 수집한 trace/metric/error 데이터를 받아 **Elasticsearch로 전달**해주는 브로커 역할을 한다.  

즉, `APM agent` → `APM server` → `Elasticsearch` → `Kibana` 순으로 연결된다.

| 역할                   | 설명                                                 |
| -------------------- | -------------------------------------------------- |
| **데이터 수신**           | 각 언어의 APM Agent가 전송하는 JSON 기반 트레이스 데이터 수신          |
| **검증 및 필터링**         | 데이터 포맷 및 필드 검증, 불필요한 필드 제거                         |
| **변환 및 전처리**         | OpenTelemetry, Jaeger 등의 trace 포맷을 Elastic 포맷으로 변환 |
| **Elasticsearch 전송** | 처리된 데이터를 Elasticsearch 인덱스로 전송 (`apm-*` 패턴)        |
| **보안 설정**            | API Key, secret token, TLS 등으로 인증 관리               |

<br>

---

<br>

# <font color="#8db3e2">APM 설치/셋팅하기</font>

<br><br>

## docker-compose.yml 수정


1. **apm-server**를 기존 docker-compose.yml 파일에 추가한다.
2. **Spring**셋팅에 필요한 매개변수도 추가해준다.
3. **Kibana**에 elastic APM활성화에 필요한 설정을 추가한다.

```yml

  # APM Server 추가
  apm-server:
    image: docker.elastic.co/apm/apm-server:7.11.1
    container_name: apm-server
    environment:
      - output.elasticsearch.hosts=["elasticsearch:9200"]
      - apm-server.host=0.0.0.0:8200
      - apm-server.frontend.enabled=true
      - apm-server.frontend.rate_limit=100000
      - apm-server.read_timeout=1m
      - apm-server.shutdown_timeout=2m
      - apm-server.write_timeout=1m
      - logging.level=info
    ports:
      - "8200:8200"
    networks:
      - elk
    depends_on:
      - elasticsearch

  # Spring app에 필요한 매개변수 추가	
  spring:
    image: ghcr.io/anonichat/app/anonichat
    expose:
      - "8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
		...생략
		ELASTIC_APM_SERVICE_NAME=anonichat-app          # APM에서 표시될 서비스명
		ELASTIC_APM_SERVER_URLS=http://apm-server:8200  # APM 서버 주소
		ELASTIC_APM_APPLICATION_PACKAGES=com.anonichat  # 모니터링할 패키지
		ELASTIC_APM_ENVIRONMENT=docker                  # 환경 구분
		ELASTIC_APM_LOG_LEVEL=INFO                      # 로그 레벨
		ELASTIC_APM_ENABLE_LOG_CORRELATION=true         # 로그 연관성 활성화
    depends_on:
      - elasticsearch
      - mysql
    networks:
      - elk
      - data

# Kibana APM 활성화에 필요한 매개변수 추가
  kibana:
    image: docker.elastic.co/kibana/kibana:7.11.1
    ports:
      - "5601:5601"
    networks:
      - elk
    environment:
      ... 생략
      - xpack.apm.ui.enabled=true # Kibana에서 APM 메뉴 활성화
    depends_on:
      - elasticsearch
```

<br>
<br>

## (Spring app) dockerfile 수정

```yml
# apm agent jar 추가
RUN curl -o /app/elastic-apm-agent.jar \  
    https://repo1.maven.org/maven2/co/elastic/apm/elastic-apm-agent/1.28.4/elastic-apm-agent-1.28.4.jar

# apm agent 실행 명령어 추가
ENTRYPOINT ["java", "-javaagent:/app/elastic-apm-agent.jar", "-jar", "/app/AnoniChatApp.jar"]
```

<br><br>

## (Spring app) build.gradle 수정

```java
# dependency 추가

// Elastic APM Agent (프로그래밍 방식 연결용)
implementation 'co.elastic.apm:apm-agent-attach:1.50.0'

// APM Spring Boot Starter (자동 설정 - Spring Boot 3.x 호환)
implementation 'co.elastic.apm:elastic-apm-spring-boot-starter:1.50.0'

// APM과 로그 연동 (ECS 로그 형식)
implementation 'co.elastic.logging:logback-ecs-encoder:1.6.0'
```

<br>
<br>

## (Spring app) application.properties 수정

- 위 docker-compose.yml 파일에서 설정한 매개변수를 삽입하여 apm설정

```yml
# APM 설정
elastic.apm.service-name=${ELASTIC_APM_SERVICE_NAME:anonichat-app}
elastic.apm.server-urls=${ELASTIC_APM_SERVER_URLS:http://localhost:8200}
elastic.apm.application-packages=${ELASTIC_APM_APPLICATION_PACKAGES:Anoni}
elastic.apm.environment=${ELASTIC_APM_ENVIRONMENT:local}
elastic.apm.enabled=${ELASTIC_APM_ENABLED:true}
elastic.apm.transaction-sample-rate=${ELASTIC_APM_SAMPLE_RATE:1.0}
```


<br>
<br>

## (Spring app) logback-spring.xml 수정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

	... 생략

    <!-- ECS 형식 appender (Elastic Stack 최적화, APM 완벽 연동) -->
    <appender name="ECS_LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>logstash:5000</destination>  <!-- 기존 포트 5000 사용 -->
        <encoder class="co.elastic.logging.logback.EcsEncoder">
            <!-- APM 서비스명 설정 -->
            <serviceName>${elastic.apm.service-name:-anonichat-app}</serviceName>
            <!-- APM 트레이스 정보 자동 포함 -->
            <includeMarkers>true</includeMarkers>
            <includeMdc>true</includeMdc>
        </encoder>
        <keepAliveDuration>5 minutes</keepAliveDuration>
    </appender>

	... 생략

    <!-- APM 관련 로그 레벨 설정 -->
    <logger name="co.elastic.apm" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>

</configuration>
```

#### 변경점 
> 로그의 데이터 형식을 *json* 형식  →  *ecs* 형식으로 변경

<br>

>[!info] **ECS(Elastic Common Schema)란?** 
 Elasticsearch에서 정의한 표준화된 로그 데이터 형식

- ECS형식의 예
```json
# 일반 json 형식
{
  "timestamp": "2025-06-15T14:30:15.123Z",
  "level": "INFO",
  "message": "사용자 로그인",
  "userId": "user123",
  "ip": "192.168.1.100",
  "userAgent": "Chrome/91.0"
}

# ECS 형식
{
  "@timestamp": "2025-06-15T14:30:15.123Z",
  "log": {
    "level": "INFO"
  },
  "message": "사용자 로그인",
  "user": {
    "id": "user123"
  },
  "client": {
    "ip": "192.168.1.100"
  },
  "user_agent": {
    "original": "Chrome/91.0"
  },
  "service": {
    "name": "anonichat-app"
  },
  "trace": {
    "id": "abc123"
  },
  "transaction": {
    "id": "def456"
  }
}
```

#### 장점

- 모든 서비스에서 표준화된 필드명을 가진다.
- kibana 대시보드와 호환성이 좋다.
- apm과 자동 연동이 된다.

<u>일반 json 형식보다 확장성, 호환성이 좋은 ECS로 로그 형식을 변경했다.</u>

---

<br>
<br>

# <font color="#8db3e2">설정 적용 및 테스트</font>


<br>


- 테스트
![[do-messenger_screenshot_2025-06-19_14_08_30.png]]  
## kibana에서 apm이 활성화 되었음을 볼 수 있다.


---

<br>

# + ERROR **Slack**연동하여 알림처리 하기

<br>
 

## 알림(Alerting) Tool 종류

### 🔔 Alerting Tool 종류 비교

| 항목       | ElastAlert                                                | Logstash Alerting                            | Kibana Watcher                                                      |
| -------- | --------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------- |
| **설명**   | Elasticsearch 기반 조건 탐지 및 알림 전송                            | Logstash 처리 파이프라인에서 조건 매칭 시 알림 전송            | Kibana 내 Watch 정의를 통한 모니터링 및 Alert                                  |
| **설정방식** | `YAML` 설정 파일                                              | Logstash 파이프라인 설정 파일                         | Kibana GUI 또는 JSON DSL                                              |
| **장점**   | - 간단한 설정  <br>- ES 쿼리 직접 사용  <br>- 오픈소스                   | - 로그 수집과 알림을 동시에 처리  <br>- *중간 저장 없이 실시간 처리* | - UI 기반 설정  <br>- ELK 통합 경험 우수  <br>- 시각화 및 경보 연동 용이                |
| **단점**   | - Python 기반 설치 필요  <br>- *독립 실행 필요*  <br>- 대량 데이터 시 성능 저하 | - 알림 기능이 제한적  <br>- 조건 로직 복잡 시 관리 어려움        | - X-Pack Gold 이상 필요  <br>- 고급 조건 설정 시 DSL 복잡  <br>- 외부 연동 제약 / *유료* |
| **기술요건** | Python 2.7/3.x, 독립 실행                                     | Logstash filter + output 설정                  | Elasticsearch Watcher (X-Pack 포함)                                   |
| **활용 예** | 보안 이벤트 탐지, 이상 로그 패턴 경보                                    | 수집 즉시 Slack 메시지 전송                           | 특정 지표가 임계치 초과 시 Email 전송                                            |
- 다음과 같은 특징들을 고려했을 때, <font color="#8db3e2">Logstash Alerting</font> 를 선택하였다.
- 현재에도 인스턴스의 리소스 과부하 문제가 있기 때문에, 가벼운 녀석을 골랐다.
1. ElastAlert은 무겁고, 독립 실행이 필요하기 때문에 리소스 과부하를 피할 수 없다.
2. Kibana Watcher는 무료 라이센스에서의 제한적인 기능과, 고급 조건이 불가하여 추적 및 디버기에 한계가 있다.

<br>

Logstash는 현재 모두 구현되어 있는 상태이고 설정파일에 몇 줄만 추가하면 알림 설정을 마칠 수 있다.

---

<br>

## 1. Slack *WebhookUrl* 발급

<br><br>


#### 1. slack 챗팅방 개설 후 `설정` → `통합` → `앱` → `앱 추가` 접속

![[do-messenger_screenshot_2025-06-23_11_43_48.png|550]]    

<br>

#### 2. `incoming webhooks` 설치

![[do-messenger_screenshot_2025-06-23_11_43_55.png|775]]

<br>

#### 3. 채팅방을 설정하고 웹훅 추가

![[do-messenger_screenshot_2025-06-23_11_44_05.png|800]]
- 웹훅 url을 발급받는다.


#### 발급 받은 웹훅 url을 이용하여 logstash.conf를 수정하면 된다.

<br>
<br>

## `logstash.conf` 수정

````python
input {
    beats {
        port => 5044
    }
    tcp {
        port => 5000
        codec => json_lines
        type => "main_log"
    }

}

filter {
    if [type] == "main_log" {
        if [log.level] in ["ERROR", "INFO", "FATAL"] { # INFO는 현재 기능 테스트를 위해 넣음
            mutate {
                add_field => { "alert_needed" => "true" }
            }
        }
    }
}

output {
	# 5000번 포트로 오는 모든 알림은 `main_log로 들어와 elastic에 저장`
    if [type] == "main_log" {
        elasticsearch {
            hosts => ["http://elasticsearch:9200"]
            index => "main_log"
        }
    }

######################## 추가 부분 ##########################
	# ["ERROR", "INFO", "FATAL"] 에 해당 하는 알림은 슬렉으로 output 처리
    if [alert_needed] == "true" { # error 레벨 발생 시 알려줌
        http {
            url => "https://hooks.slack.com/services/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"  # output 플러그인 사용
            http_method => "post" # Slack Webhook URL로 `POST` 요청 전송.
            content_type => "application/json"
            format => "json"
            mapping => { # 메시지는 `%{}` 문법으로 필드 값을 문자열에 삽입:
                "text" => 
				"🚨 *AnoniChat 에러 발생!*```
                레벨: %{[log.level]}
				시간: %{[@timestamp]}
				스레드: %{[process.thread.name]}
				로거: %{[log.logger]}
				메시지: %{message}
				Trace ID: %{[trace.id]}
				서비스: %{[service.name]}
				```"
                "channel" => "#anonichat-error-log"
            }
        }
    }
    
##########################################################
}
````

<br>

- 이후에 logstash 재시작.

```
docker-compose restart logstash
```

---

<br>

## <font color="#8db3e2">테스트</font>

<br>

- anonichat.world(*APP*) 로 접속해서 -info 로그가 두 번 찍히는지 테스트

```java
@GetMapping(GlobalURL.MAIN_URL)  
public ModelAndView mainView() {  
    log.info("[MainController Log] mainView 접속 TEST");  
  
    List<TestEntity> list = testRepository.findAll();  
    log.info("[MainController Log] test_table data: {}", list);  
  
    return new ModelAndView("main");  
}
```


![[do-messenger_screenshot_2025-06-23_12_07_38.png|1125]]
- INFO 로그가 정상 출력되는 모습. (log.info로 테스트)

<u>이후 *logstash.conf*에서 INFO를 빼고 error 레벨만 알람이 울리도록 조정을 다시 해줬다.</u>