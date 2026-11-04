# 🏧 ATMS - JEUS 매니지드 서버 6개 구성기(게이트웨이를 걷어낸 이유)

#프로젝트 #인프라 #JEUS #WebtoB #WAS #SpringBoot #AI #ClaudeCode

---

<br>

지난 글까지는 업체 개발 서버에서 docker compose로 7개 서비스를 띄우고, CI/CD와 저널 검색 파이프라인을 붙인 이야기였다.

## ▶ [[🏧 ATMS - ELK Stack 구축(Kafka 기반 저널 스트리밍)]]

이번 글부터는 무대가 **은행 개발 서버**로 옮겨간다. 그리고 첫 번째 질문은 의외로 단순했다 — **"이 6개 모듈을 은행 WAS에 어떻게 올리지?"**

---

<br>

# <font color="#76923c">배경 — 컨테이너가 아니라 은행 표준 WAS 위로</font>

<br>

업체 개발 환경에서는 모듈마다 컨테이너 하나씩, 내장 톰캣으로 떠 있었다. 그런데 은행 개발 서버는 조건이 완전히 달랐다.

- 은행 표준 WAS인 **JEUS**와 웹서버 **WebtoB**가 이미 설치되어 있고, 백엔드는 그 위에서 돌아야 한다.
- **폐쇄망**이라 외부 레지스트리나 Maven 저장소에 접근할 수 없다.
- 서버는 **2코어**이고, JEUS 설치·서버 생성·재기동은 **WAS 담당**의 영역이다.

<br>

>[!info] JEUS / WebtoB 한 줄 정리
>|구분|역할|익숙한 것에 비유하면|
>|---|---|---|
>|JEUS|WAS (서블릿 컨테이너)|Tomcat|
>|WebtoB|웹서버 (정적 파일 + WAS 연동)|Apache / nginx|
>|JSV|WebtoB ↔ JEUS 전용 연결|AJP / 리버스 프록시|
>
> 특이한 점은 JSV 연결의 방향이다. **JEUS 서버가 WebtoB의 JSV 포트로 접속해 온다.** nginx처럼 웹서버가 WAS를 찾아가는 구조가 아니다.

그래서 빌드 산출물부터 바꿨다. 실행형 JAR(`bootJar`) 대신 **WAR**(`bootWar -Pjeus`)를 만들고, 모든 `Application` 클래스가 `SpringBootServletInitializer`를 상속하게 했다. `-Pjeus`를 줄 때만 WAR로 바뀌기 때문에 로컬의 `bootRun`과 기존 JAR 빌드는 그대로다.

```groovy
// 루트 build.gradle — -Pjeus 로 빌드할 때만 WAR 로 전환
pluginManager.withPlugin('org.springframework.boot') {
    if (project.hasProperty('jeus')) {
        apply plugin: 'war'
        dependencies {
            // 내장 톰캣은 WAR 에서 제외 (JEUS 가 서블릿 컨테이너 역할)
            providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
        }
        tasks.named('bootWar') {
            from(rootProject.file('jeus/web.xml')) {
                into 'WEB-INF'
                expand(profile: 'jeus', moduleName: project.name)
            }
        }
    }
}
```

>[!tip] 프로파일은 JVM 옵션이 아니라 web.xml로
> 한 JVM에 WAR 여러 개가 올라갈 수 있으니, `-Dspring.profiles.active`를 JVM 옵션으로 넣으면 모든 앱이 같은 값을 공유해버린다. 그래서 각 WAR의 `WEB-INF/web.xml`에 `spring.profiles.active`를 context-param으로 넣어 **WAR마다 프로파일을 들고 다니게** 했다.

```xml
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee" version="5.0" metadata-complete="false">
    <display-name>${moduleName}</display-name>
    <context-param>
        <param-name>spring.profiles.active</param-name>
        <param-value>${profile}</param-value>
    </context-param>
</web-app>
```

---

<br>

# <font color="#76923c">1차 구성 — 서버 1개 + 컨텍스트 6개 + 게이트웨이</font>

<br>

처음에는 가장 단순해 보이는 방법을 택했다. JEUS 서버(=JVM) **하나**에 WAR 6개를 올리고, **컨텍스트 경로**로 구분하는 방식이다.

>[!info] 1차 구성의 컨텍스트 배치
>|모듈|컨텍스트|
>|---|---|
>|atms-web|`/`|
>|atms-journal|`/journal`|
>|atms-batch|`/batch`|
>|atms-file|`/file`|
>|atms-if-internal|`/ifin`|
>|atms-if-external|`/ifex`|

문제는 **바꿀 수 없는 주소**들이었다. ATM 에이전트는 `/api/journal/...`, `/atmf/...` 같은 고정 경로로 저널과 파일을 보내고, 대내·대외 기관도 정해진 경로로 호출한다. 이 요청들은 `/journal` 같은 컨텍스트를 모르기 때문에 **전부 루트(`/`) = atms-web으로 떨어진다.**

그래서 atms-web 맨 앞에 **게이트웨이 필터**를 하나 뒀다. 경로 접두어를 보고, 같은 JEUS 안의 해당 컨텍스트로 HTTP 요청을 다시 보내는 방식이다.

```c
graph TD
	A[ATM 에이전트 POST /api/journal/...]
	B[WebtoB]
	C[JEUS 서버 1개 - 컨텍스트 / = atms-web]
	D[ModuleGatewayFilter - 접두어 매칭]
	E[같은 JEUS 의 /journal 컨텍스트로 HTTP 재요청]
	F[atms-journal 처리]

	A --> B --> C --> D --> E --> F
```

```yaml
gateway:
  enabled: true
  routes:
    - prefix: /api/journal
      target: ${jeus.base-url}/journal
    - prefix: /api/batch
      target: ${jeus.base-url}/batch
    - prefix: /atmf
      target: ${jeus.base-url}/file
    # ... /api/agent, /api/file, /api/if/internal, /api/if/external
```

Tomcat에 올려 검증까지 마쳤고, 동작 자체는 문제없었다. 다만 이 구조에서 **컨텍스트 경로 때문에 생긴 버그 두 개**를 만났다.

>[!bug] 함정 1 — /journal 아래에서 HMAC 검증이 조용히 빠졌다
> 저널 수신을 검증하는 HMAC 필터가 `getRequestURI()`로 `/api/journal` 여부를 판단하고 있었다. 컨텍스트가 `/`일 때는 맞았지만, JEUS에서는 URI가 `/journal/api/journal/...`이 되면서 **매칭 실패 → 검증 자체를 건너뛰었다.** 에러가 아니라 "통과"로 나타나서 더 위험했다. 경로를 비교할 때는 `getContextPath()`를 뺀 경로로 판단하도록 고쳤다.

>[!bug] 함정 2 — 대상 모듈이 꺼져 있으면 사용자가 로그아웃됐다
> 게이트웨이가 연결에 실패하면 `sendError(502)`를 호출했는데, `/error`로 재디스패치된 요청에는 인증 정보가 없어서 Spring Security가 이를 **401로 바꿔버렸다.** 프론트는 401을 받으면 로그인 화면으로 보내니, 모듈 하나가 죽으면 사용자가 로그아웃되는 셈이었다. `sendError` 대신 502 상태와 JSON 본문을 직접 써서 응답하도록 바꿨다.

---

<br>

# <font color="#76923c">게이트웨이를 직접 재 보니 — 성능과 스레드 교착</font>

<br>

"게이트웨이 하나 더 거치는 게 그렇게 큰 비용일까?" 감으로 판단하지 않고 Claude Code와 함께 **직접 재보기로** 했다. 은행 JEUS에서 부하 테스트를 할 수는 없어서, Tomcat 10.1에 web·file 두 모듈을 올려 참고치로 측정했다.

>[!info] 성능 실측 (Tomcat 10.1, 4코어 — JEUS 아님, 참고치)
>|항목|직접 호출|게이트웨이 경유|차이|
>|---|---|---|---|
>|작은 JSON 순차 300회 (p50)|5.3ms|7.9ms|건당 약 +2.6ms|
>|처리량 (동시 20)|690 req/s|518 req/s|약 -25%|
>|50MB 다운로드|596ms|883ms|약 -32%|

화면 수준의 트래픽에서는 체감하기 어려운 차이다. 하지만 이 시스템에는 **수천 대의 ATM이 계속 보내는 저널**과 **대용량 파일 다운로드**가 있고, 2코어 서버에서 전달 요청은 CPU를 약 1.5배 쓴다.

그리고 진짜 문제는 숫자보다 **구조**에 있었다.

>[!danger] 게이트웨이 요청은 스레드를 2개 쓴다
> atms-web의 스레드가 대상 모듈의 응답을 기다리는 동안, 대상 모듈도 요청을 처리할 스레드가 필요하다. 둘이 **같은 스레드 풀**에서 나오면 서로를 기다리며 멈출 수 있다.

실제로 재현해봤다. 스레드 풀을 4개로 줄이고 동시 16건씩 32건을 보냈다.

>[!failure] 재현 결과 — 직접 호출은 전부 성공, 게이트웨이는 전부 타임아웃
>|호출 방식|결과|
>|---|---|
>|직접 호출|32건 모두 성공 (0.5초)|
>|게이트웨이 경유|**32건 모두 타임아웃** — 읽기 타임아웃(120초)까지 서버 전체 무응답|
>
> 스레드 덤프를 떠보니 4개 스레드가 모두 `ModuleGatewayFilter`에서 응답을 기다리고 있었고, 정작 대상 앱은 스레드를 하나도 받지 못하고 있었다.

```c
graph TD
	A[요청 4건 도착 - 스레드 4개 모두 사용]
	B[각 스레드가 게이트웨이에서 /journal 로 재요청]
	C[재요청이 같은 풀에서 빈 스레드를 기다림]
	D[빈 스레드 없음 - 아무도 끝나지 않음]

	A --> B --> C --> D
	D -->|타임아웃까지 대기| A
```

JEUS에서는 외부 요청(WebtoB 커넥터)과 재요청(HTTP 리스너)이 서로 다른 풀을 쓰는 구조라 당장 터질 가능성은 낮았다. 하지만 **"풀 설정에 따라 서버 전체가 멈출 수 있는 구조"** 를 운영까지 가져가고 싶지는 않았다.

---

<br>

# <font color="#76923c">"JEUS 6개"는 무엇을 뜻하는가</font>

<br>

결정적인 계기가 하나 더 있었다. 서버를 점검해보니 6개 앱 중 **1개만 등록**되어 있었고, 나머지 5개 등록은 어차피 WAS 담당에게 요청해야 했다. **어차피 요청할 거라면 구성 자체를 다시 정하자**고 판단했다.

그런데 "모듈마다 포트를 따로 갖게 해 주세요"를 JEUS의 언어로 바꾸려니 용어부터 헷갈렸다. 그래서 Claude Code에게 질문을 하나씩 쪼개서 던지며 개념부터 맞췄다.

>[!tip] 질문을 쪼개가며 개념부터 맞췄다
>|순서|질문|정리된 답|
>|---|---|---|
>|1|JEUS 하나에서 6개를 각자 포트로 띄울 수 있나?|서버 1개(=JVM 1개)에서는 WAR가 포트를 갖지 않는다. 서버의 리스너를 공유하고 컨텍스트로 구분할 뿐이다|
>|2|"JEUS 6개"가 가리킬 수 있는 구성은?|① 서버 1개 + 리스너 6개 ② 도메인 1개 + 매니지드 서버 6개 ③ JEUS(도메인) 6개 — 차이의 핵심은 **JVM 수**|
>|3|내장 톰캣 6개를 띄우던 업체 구성은 셋 중 무엇과 같나?|구조(JVM 6개·포트 6개·장애 격리)는 ②와 같다|
>|4|톰캣과 JEUS는 같은 개념 아닌가?|맞다 — 둘 다 서블릿 컨테이너다. 차이는 JEUS 쪽에 관리 층(DAS·NodeManager)이 붙는다는 것뿐|

JEUS의 구조를 그려보면 이렇다.

```text
물리 서버 (2코어)
└─ JEUS 설치
   └─ 도메인 (domain.xml 하나)
      ├─ DAS               ← 관리 전용 JVM. 설정 보관·배포·서버 기동/정지 명령
      ├─ NodeManager       ← 서버 프로세스를 띄우고 감시하는 데몬
      └─ 매니지드 서버 × N  ← 실제 앱이 도는 JVM. 각자 리스너(포트)·힙·로그
         └─ 애플리케이션 (WAR)
```

>[!info] 세 가지 "JEUS 6개" 비교
>|구성|JVM 수|판단|
>|---|---|---|
>|① 서버 1개 + 리스너 6개|1|포트만 나뉘고 장애 격리는 없음, 지원 여부도 미확인|
>|② 도메인 1개 + 매니지드 서버 6개|6 (+DAS)|**채택**|
>|③ JEUS(도메인) 6개|12|관리 층이 6벌 — 2코어 서버에 부적합|

>[!info] 내장 톰캣과 JEUS 매니지드 서버의 대응
>|내장 톰캣 (로컬)|JEUS|
>|---|---|
>|`java -jar atms-journal.jar` 프로세스 1개|매니지드 서버 1개 (JVM)|
>|`server.port=8081`|서버의 HTTP 리스너 포트|
>|앱이 톰캣을 품음 (JAR)|서버가 앱을 품음 (WAR 등록)|
>|프로세스를 직접 실행·정지|DAS·NodeManager가 서버들을 관리|

결국 **"매니지드 서버 6개" = "내장 톰캣 6개"** 와 실행 단위가 같다. 돌고 돌아 업체 개발 환경의 구조(모듈당 프로세스 하나)로 돌아가는 셈이었다.

---

<br>

# <font color="#76923c">최종 구성 — 매니지드 서버 6개, 컨텍스트는 모두 /</font>

<br>

```c
graph TD
	PC[운영자 PC 브라우저] --> W[WebtoB]
	W -->|/index.html, /assets/| S[정적 파일 - 화면]
	W -->|/api/journal/, /api/agent/| J[journal 서버 :8081]
	W -->|/api/batch/| B[batch 서버 :8082]
	W -->|/api/file/, /atmf/| F[file 서버 :8083]
	W -->|/api/if/internal/| I[if-internal 서버 :8084]
	W -->|/api/if/external/| E[if-external 서버 :8085]
	W -->|나머지 /api/| WEB[web 서버 :8080]
```

- 포트는 **로컬 개발과 똑같이 8080~8085**로 맞췄다. 모듈 간 호출 주소 체계가 로컬과 같아져서 JEUS 전용 경로 처리가 대부분 필요 없어졌다.
- 컨텍스트는 전부 `/`이고, 게이트웨이는 **껐다.** 경로 분배는 WebtoB가 각 서버로 직접 한다.
- WebtoB 경로 규칙은 **구체적인 경로를 먼저** 적는다. (`/api/journal/`이 `/api/`보다 앞)
- ATM 에이전트는 WebtoB를 거치지 않고, 내부 L4를 통해 journal·file 서버로 바로 들어온다.

<br>

>[!info] ②를 고른 이유와 감수하는 것
>|얻는 것|감수하는 것|
>|---|---|
>|모듈별 장애 격리 (한 모듈의 OOM·스레드 고갈이 다른 모듈로 번지지 않음)|JVM 6개분 메모리·CPU → 서버당 힙을 512m~768m로 작게|
>|모듈만 따로 재기동 (이전에는 6개가 함께 내려감)|서버 생성·WebtoB 커넥터 등록 같은 WAS 담당 작업|
>|로컬과 같은 주소 체계 (8080~8085, 컨텍스트 /)|서버마다 로그·재기동 메뉴·모니터링이 따로|
>|게이트웨이 제거 → 지연·처리량 손해와 교착 위험이 사라짐|WebtoB 경로 규칙이 6개 서버로 늘어남|

---

<br>

# <font color="#76923c">바뀐 것 — 소스는 yml 몇 줄</font>

<br>

구성은 크게 바뀌었지만 소스 변경은 의외로 작았다. 모듈 간 호출 주소를 **한 곳**에 모으고, 각 모듈이 그 값을 참조하게 했다.

**① 공통 설정 `jeus-common.yml`** — 모듈별 주소를 한 곳에
```yaml
jeus:
  host: 10.0.20.15        # 리스너가 실제로 LISTEN 하는 주소
  url:
    web:     http://${jeus.host}:8080
    journal: http://${jeus.host}:8081
    batch:   http://${jeus.host}:8082
    file:    http://${jeus.host}:8083
    ifin:    http://${jeus.host}:8084
    ifex:    http://${jeus.host}:8085
```

**② 모듈별 `application-jeus.yml`** — 게이트웨이를 끄고 주소는 참조만
```yaml
spring:
  config:
    import: classpath:jeus-common.yml

# 레거시 에이전트 .do 경로 → atms-journal
agent:
  proxy:
    journal-base-url: ${jeus.url.journal}

# 모듈 경로는 WebtoB 가 각 서버로 바로 보낸다 → atms-web 이 전달하지 않음
gateway:
  enabled: false
```

**③ 배포 스크립트** — 모듈 → 포트, 재기동은 그 서버만
- 기동 확인은 `http://<서버>:<포트>/v3/api-docs`가 200인지로 본다. (actuator health는 DB가 붙기 전에는 503이라 대신 사용)
- Jenkins 배포 뒤 재기동도 **배포한 모듈의 서버만** 하나씩 한다. 재기동 수단이 아직 없는 모듈은 파일만 교체하고 빌드를 UNSTABLE로 표시해서 사람이 확인하게 했다.

>[!warning] 127.0.0.1로는 안 된다
> 이전 구성에서 리스너가 서버 IP에만 LISTEN하고 있어서, `127.0.0.1`로 상태를 확인하던 스크립트가 항상 `000`을 받은 적이 있다. **모듈 간 호출 주소와 헬스체크 주소는 리스너가 실제로 LISTEN하는 주소와 같아야 한다.** `ss -ltn`으로 먼저 확인하자.

---

<br>

**<font color="#76923c">결론</font>**
- 게이트웨이 최적화 X
- 서버 1개 + 컨텍스트 6개 유지 X
- <u>**모듈마다 JVM 하나 — 매니지드 서버 6개 + WebtoB 경로 분배 O**</u>

처음 구성은 "JEUS 하나에 다 올린다"는 직관에서 출발했고, 그 직관을 지키려고 게이트웨이라는 우회로를 만들었다. 그런데 직접 재 보니 그 우회로가 성능과 안정성 모두에서 비용을 만들고 있었다. 결국 답은 **업체 개발 환경과 같은 구조(모듈당 프로세스 하나)를 WAS의 언어로 옮기는 것**이었다.

AI와의 협업에서 가장 도움이 된 건 정답을 받는 것이 아니라, **"JEUS 6개"처럼 모호한 말을 질문 단위로 쪼개 개념부터 맞춘 것**이었다. 용어가 정리되자 WAS 담당에게 보낼 요청서도 한 장으로 정리됐다.

---

<br>

다음에는 인터넷이 막힌 폐쇄망에서 Gitea + Jenkins로 CI/CD를 다시 구성한 이야기를 정리해보겠다.
