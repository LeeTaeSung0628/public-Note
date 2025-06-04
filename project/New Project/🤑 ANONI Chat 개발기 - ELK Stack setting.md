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

# <font color="#76923c">Ubuntu server 셋팅</font>

<br>

## Kibana 사용을 위한 셋팅