# 🏧 ATMS - ELK Stack 구축(Kafka 기반 저널 스트리밍)

#프로젝트 #개발 #인프라 #Kafka #Elasticsearch #Kibana #AI #ClaudeCode

---

<br>

지난 글 말미에 다음엔 모니터링(Prometheus + Grafana) 이야기를 하겠다고 예고했었는데, 그 전에 최근 작업한 **전자저널 실시간 검색 파이프라인** 구축이 먼저 마무리돼서 이 글부터 정리한다.

## ▶ [[🏧 ATMS - CICD 트러블슈팅(CORS 403편)]]

---

<br>

# <font color="#76923c">문제 — DB엔 쌓이는데, 검색할 방법이 SQL뿐이다</font>

<br>

ATM 기기(5,500대)가 보내는 전자저널(거래/상태/장애/마감/시재 등)은 원래도 Oracle DB(`T_JOURNAL`, `T_JOURNAL_DATA`)에 잘 쌓이고 있었다. 문제는 **운영 현황을 실시간으로 들여다볼 방법이 SQL 조회뿐**이었다는 것 — 특정 기기가 언제부터 수신이 끊겼는지, 장애 저널이 어느 기기에 몰리는지를 보려면 매번 쿼리를 짜야 했다.

그래서 저널이 들어오는 즉시 검색 가능한 형태로 색인하고, 운영자가 볼 수 있는 대시보드까지 붙이기로 했다.

---

<br>

# <font color="#76923c">설계 1 — 왜 Logstash가 아니라 Kafka인가</font>

<br>

>[!tip] 설계 단계부터 Claude Code와 논의했다
> "검색 가능하게 만들자"까지는 쉬웠지만 그 다음이 갈림길이었다. **로그 기반**(Logstash가 애플리케이션 로그를 tail)으로 갈지, **이벤트 기반**(애플리케이션이 직접 발행)으로 갈지 — 각 방식의 트레이드오프를 Claude Code와 하나씩 짚어보고, 이 프로젝트의 제약 조건에 맞춰 골랐다.

제약 조건은 세 가지였다.

1. **원본은 반드시 DB**여야 한다 — 검색 인덱스는 어디까지나 사본이어야 한다.
2. **기기별 순서 보장**이 필요하다 — 같은 기기의 저널이 뒤섞여 색인되면 "최근 수신 시각" 같은 지표가 무의미해진다.
3. **저널 수신 경로(24/7 미션크리티컬)에 부가 기능이 영향을 주면 안 된다.**

<br>

>[!info] 로그 기반 vs 이벤트 기반
>|구분|로그 기반 (Logstash가 로그 tail)|이벤트 기반 (이번 선택)|
>|---|---|---|
>|원본 위치|로그 파일(회전·유실 위험)|DB(항상 원본)|
>|데이터 구조|텍스트 파싱 필요|이미 구조화된 JSON|
>|기기별 순서 보장|사실상 어려움|Kafka 파티션 키로 보장|
>|수신 경로 영향|거의 없음(완전 비동기 tail)|**있음** — 발행 코드가 수신 경로에 추가됨|

로그 기반이 "수신 경로 무영향" 항목에서는 더 유리하다. 그런데도 이벤트 기반을 택한 이유는, 저널이 **이미 구조화된 비즈니스 이벤트**이지 텍스트 로그가 아니었기 때문이다 — 로그로 한번 풀어썼다가 Logstash로 다시 파싱하는 건 우회였다.

대신 이벤트 기반의 유일한 약점("수신 경로 영향")은 다음 설계로 정면 대응하기로 했다.

---

<br>

# <font color="#76923c">설계 2 — 수신 경로 무영향을 코드로 강제하기</font>

<br>

원칙은 "부가 기능이 핵심 기능을 절대 건드리면 안 된다"는 것. 이걸 문서로만 남기지 않고 네 가지 장치로 코드에 박아 넣었다.

**① 완전 opt-in — 플래그 두 개, 기본값 전부 false**

```java
@ConfigurationProperties(prefix = "atms.stream")
public class JournalStreamProperties {
    private final Kafka kafka = new Kafka();                 // kafka.enabled = false (기본)
    private final Elasticsearch elasticsearch = new Elasticsearch(); // enabled = false (기본)
}
```

**② 관련 컴포넌트 전부 `@ConditionalOnProperty`로 게이팅**

토픽 생성(`JournalStreamTopicConfig`), 인덱스 템플릿 등록(`JournalEsIndexInitializer`), 컨슈머(`JournalEsIndexConsumer`) — 셋 다 플래그가 꺼져 있으면 스프링 컨텍스트에 아예 존재하지 않는다.

**③ DB 커밋 이후에만 발행, 그리고 실패해도 예외를 던지지 않는다**

```java
// JournalDispatchService
txRequired.executeWithoutResult(tx -> doDispatchInTx(key, msg));
// DB 적재 성공 후에만 스트림 발행(ES 색인용) — 발행 실패는 수신 응답에 영향 없음
eventPublisher.publishStored(key, msg);
```

```java
// JournalEventPublisher.publishStored()
if (!props.getKafka().isEnabled()) return;           // 꺼져 있으면 즉시 no-op
...
template.send(topic, kafkaKey, payload)
        .whenComplete((result, ex) -> {
            if (ex != null) log.warn("journal event publish failed ...: {}", ex.toString());
            // 예외를 여기서 절대 다시 던지지 않는다
        });
```

**④ ES가 죽어있어도 앱 기동을 막지 않는다**

`JournalEsIndexInitializer`는 기동 시 인덱스 템플릿을 등록하는데, ES가 아직 안 떠 있으면 경고 로그만 남기고 그냥 넘어간다 (색인 시점에 동적 매핑으로 대체 동작).

>[!question] "그냥 try-catch로 감싸면 되는 거 아닌가?"
> Claude Code와 이 부분을 검토하면서 나온 얘기다. try-catch로 감싸는 것과, **컴포넌트 자체를 조건부로 존재하지 않게 만드는 것**은 다르다. 후자는 "이 기능을 켠 적이 없는 환경"에서는 관련 빈이 아예 생성되지 않으니, 코드 리뷰 시점에도 "이 배포에 이 기능이 관여하는가"가 설정 하나로 명확해진다.

---

<br>

# <font color="#76923c">설계 3 — 재전송돼도 중복되지 않게(멱등성)</font>

<br>

ATM 에이전트는 응답을 못 받으면 같은 저널을 재전송한다. Kafka 컨슈머가 같은 메시지를 두 번 처리할 가능성도 있다. 그래서 색인 시점에 **결정론적 문서 ID**를 만들고, `POST`가 아닌 `PUT`으로 색인하도록 했다.

```java
// JournalEsIndexConsumer
String docId = String.join("-",
        termGroupId, termId, jnlDate, jnlTime, jnlSeq);

Request request = new Request("PUT", "/" + indexName(jnlDate) + "/_doc/" + docId);
```

같은 저널이 몇 번을 다시 들어와도 도착하는 문서 ID는 항상 동일하니, 중복 색인이 아니라 **같은 문서를 덮어쓰는 것**으로 끝난다. 인덱스 이름도 "오늘 날짜"가 아니라 **저널 자체의 거래일자**(`jnlDate`)로 정해서, 뒤늦게 들어온 지난 저널도 엉뚱한 날짜의 인덱스에 쌓이지 않는다.

필드 매핑은 저널 종류(기동/개국/상태/장애/마감/시재/거래 — 총 7종)마다 구조가 달라서, `head`/`body`를 Elasticsearch의 **`flattened`** 타입으로 매핑해 스키마 걱정 없이 통째로 색인하고, 전체 payload는 `searchText`(text 타입)에 중복 저장해 Kibana에서 계좌·에러코드 전문검색이 가능하게 했다.

---

<br>

# <font color="#76923c">전체 그림</font>

<br>

```c
graph TD
	A[ATM Agent 5,500대]
	B[atms-journal 수신 API]
	C[Oracle DB 적재 - T_JOURNAL]
	D[JournalEventPublisher - fire-and-forget]
	E[Kafka topic atms.journal.received - 파티션 12]
	F[JournalEsIndexConsumer]
	G[Elasticsearch 일별 인덱스]
	H[Kibana 대시보드]

	A --> B --> C --> D --> E --> F --> G --> H
```

- 파티션 키는 `termGroupId:termId` — 같은 기기의 저널은 항상 같은 파티션으로 가서 순서가 보장된다.
- Kafka 토픽 자체는 원본이 아니라 색인용 버퍼라서 보존기간을 3일로 짧게 잡았다. (원본은 어디까지나 DB)

<br>

개발 환경은 Docker Compose 한 방으로 띄운다.

```bash
docker compose up -d kafka elasticsearch kibana atms-journal
./infra/elastic/setup-kibana.sh    # 데이터뷰(atms-journal-*) 등록
```

Kafka는 Zookeeper 없는 KRaft 단일 노드(3.9.1), ES/Kibana는 8.18.2. 전부 개발용 단일 노드 구성이고, 운영 전환 시에는 Kafka 3노드+SASL/TLS, ES 3노드+보안 활성화, 계좌번호 등 개인정보 마스킹 후 색인, 수동 삭제 대신 ILM 정책 적용이 필요하다 — 여기까진 아직 TODO로 남아있다.

---

<br>

# <font color="#76923c">실측 — 목표 TPS를 실제로 채웠는가</font>

<br>

설계만 그럴듯한 것과 실제로 부하를 견디는 것은 다른 문제라, 에이전트 프로토콜을 그대로 흉내 낸 부하 테스트 스크립트로 피크 시나리오(550대 × 기기당 10건 = 5,500건, 병렬 32)를 두 번 돌렸다.

>[!info] 부하 테스트 결과
>|시점|환경|결과|
>|---|---|---|
>|1차|로컬, Kafka/ES **off**|5,500/5,500 성공, 42초, **TPS 131건/초** (목표 하한 100/s 상회)|
>|2차|배포 서버, Kafka/ES **on**|5,500/5,500 성공, 16초, **TPS 343건/초** (목표 상한 300/s도 상회)|
>
> 2차 테스트에서 Kafka→ES 색인까지 **5,501/5,501 무손실**, dispatch 실패·publish 실패 모두 0건.

Kafka/ES를 켰는데 오히려 더 빨라진 건, 두 차례 테스트 사이에 배치 처리 경로 자체도 함께 손봤기 때문이다 — 스트리밍 파이프라인이 "얹혀서" 느려지지 않았다는 게 이번 검증의 핵심이었다.

<br>

## 부하 테스트가 찾아낸 진짜 버그 2건

성능 수치보다 부하 테스트 자체가 잡아낸 결함이 더 유용했다.

**① CASSETTE 상태 MERGE가 두 번째 저널부터 실패**

```
ORA-00001: unique constraint violated
```

같은 기기의 시재/거래 저널이 두 번째로 들어오는 순간부터 실패했다. 원인은 `MERGE` 문의 매칭 조건 — DB 컬럼은 `CHAR(5)`(공백 패딩)인데 바인드 값은 `VARCHAR`라 항상 `NOT MATCHED`로 판정되어, 매번 새로 INSERT를 시도하다 PK 충돌이 난 것.

```sql
-- 수정 전: CHAR(5) vs VARCHAR 비교라 매번 불일치
ON (T.CASSETTE_NO = :cassetteNo)

-- 수정 후: 명시적으로 CHAR(5)로 캐스팅해 비교
ON (T.CASSETTE_NO = CAST(:cassetteNo AS CHAR(5)))
```

**② 거래저널 은행코드 자리수 불일치로 인한 오버플로**

저널 스펙상 은행코드는 4자리인데, 정작 DB 컬럼은 3자리로 정의돼 있어 값이 잘려 들어가는 문제가 있었다. 스펙과 스키마 중 어느 쪽이 맞는지는 설계 검증이 더 필요한 부분이라, 우선 뒤 3자리만 취하는 정규화 함수를 추가해 오버플로부터 막았다.

```java
private static String bank3(String bkCd) {
    // 스펙(4자리) → 컬럼(3자리) 정규화. 예: "1234" → "234"
    return (bkCd != null && bkCd.length() == 4) ? bkCd.substring(1) : bkCd;
}
```

둘 다 평소 트래픽에서는 안 드러나다가, 부하 테스트로 "같은 기기가 반복해서 저널을 보내는" 상황을 재현하고 나서야 나온 결함이었다.

---

<br>

**<font color="#76923c">결론</font>**
- 검색을 위해 로그로 우회할 필요 없음 — 저널은 이미 구조화된 이벤트였다.
- 부가 기능(검색/대시보드)이 핵심 경로(저널 수신)를 건드리지 않게 만드는 건 원칙 선언이 아니라 **옵트인 플래그 + 조건부 빈 + 예외 미전파**라는 구체적 코드 패턴으로 강제해야 한다.
- 멱등성은 재전송을 막는 게 아니라, **재전송이 일어나도 결과가 같도록** 만드는 것.
- 목표 TPS는 숫자로만 정하고 끝내지 않고, 실제 부하로 검증했을 때 진짜 결함(두 건)이 나왔다.

Kibana 대시보드를 코드(PowerShell 스크립트)로 조립해 배포하는 이야기는 분량상 이번 글에서는 다루지 못했다 — 다음 기회에 따로 정리해본다.
