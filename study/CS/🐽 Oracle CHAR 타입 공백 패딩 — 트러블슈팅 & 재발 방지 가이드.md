# 🐽 Oracle CHAR 타입 공백 패딩 — 트러블슈팅 & 재발 방지 가이드

#공부 #구조 #설계 #MSA #CloudNative

---
<br />

## 목차

  

1. [현상](#1-현상)

2. [원인 분석](#2-원인-분석)

3. [로그에서 정상으로 보였던 이유](#3-로그에서-정상으로-보였던-이유)

4. [적용한 수정 내역](#4-적용한-수정-내역)

5. [재발 방지 대책](#5-재발-방지-대책)

6. [Oracle CHAR vs VARCHAR2 핵심 정리](#6-oracle-char-vs-varchar2-핵심-정리)

  

---

  

## 1. 현상  
  
| 항목             | 내용                                                    |
| -------------- | ----------------------------------------------------- |
| **발생 기능**      | 기준정보 > 코드관리 — 코드 그룹 단건 조회 (`selectGroupOne`)          |
| **증상**         | 전체 목록 조회(`selectGroupList`)는 정상, 단건 조회는 항상 데이터 없음(0건) |
| **P6SPY 로그**   | SQL·파라미터 모두 정상으로 출력됨                                  |
| **MyBatis 매핑** | 결과 객체 `null` 반환 (매핑 실패가 아닌 0건 반환)                     |

  

---

  

## 2. 원인 분석

  

### 2.1 Oracle CHAR 타입의 공백 패딩

  

Oracle의 `CHAR(n)` 컬럼은 입력값이 선언 길이보다 짧을 경우 **오른쪽에 공백을 패딩**하여 저장한다.

  

```

-- GROUP_CD 컬럼이 CHAR(5)로 선언된 경우

INSERT INTO T_CODEGROUP VALUES ('KOR', '002', '거래구분');

-- → DB에는 GROUP_CD = '002  ' (뒤에 공백 2자리 패딩되어 저장)

```

  

디버그 로그로 확인한 실제 저장값:

  

```

[DEBUG] DB 실제값 langCd='KOR' groupCd='002  ' groupName='거래구분'

                                        ↑↑ 공백 2자리 패딩

```

  

### 2.2 Oracle의 두 가지 문자열 비교 규칙

  

Oracle은 비교 대상의 데이터 타입에 따라 다른 비교 규칙을 적용한다.

  

| 비교 조합             | 적용 규칙                       | 결과 예시                         |
| ----------------- | --------------------------- | ----------------------------- |
| `CHAR = CHAR`     | **공백 패딩 비교** (blank-padded) | `'002' = '002  '` → **TRUE**  |
| `CHAR = VARCHAR2` | **비패딩 비교** (non-padded)     | `'002' = '002  '` → **FALSE** |
  

### 2.3 JDBC 파라미터는 VARCHAR2로 전송된다

  

MyBatis/JDBC는 Java `String` 타입 파라미터를 Oracle에 **VARCHAR2**로 바인딩한다.

  

```

애플리케이션: groupCd = "002"    → JDBC PreparedStatement → Oracle VARCHAR2('002')

DB 저장값:    GROUP_CD = '002  ' → CHAR(5)

```

  

비교 조합이 `CHAR = VARCHAR2`가 되므로 비패딩 비교 규칙이 적용되어:

  

```sql

-- 실제로 Oracle이 수행하는 비교

WHERE GROUP_CD = '002'

-- '002  ' (CHAR)  =  '002' (VARCHAR2)  → FALSE → 0건 반환

```

  

### 2.4 selectGroupList는 왜 정상이었나

  

`selectGroupList`는 WHERE 절이 없는 **전체 조회**이므로 파라미터 바인딩이 발생하지 않는다. CHAR 패딩 값 그대로 반환될 뿐, 비교 연산이 없어 문제가 드러나지 않았다.

  

```sql

-- 목록 조회: WHERE 없음 → 비교 없음 → 패딩값 그대로 반환

SELECT LANG_CD, GROUP_CD, GROUP_NAME FROM T_CODEGROUP ORDER BY LANG_CD, GROUP_CD

-- → 정상 동작

  

-- 단건 조회: CHAR 컬럼을 VARCHAR2 파라미터와 비교 → 0건

SELECT ... FROM T_CODEGROUP WHERE GROUP_CD = '002'

-- → 0건 반환 (DB에는 '002  '로 저장되어 있음)

```

  

---

  

## 3. 로그에서 정상으로 보였던 이유

  

### 3.1 P6SPY는 바인딩 직전 값을 로깅한다

  

P6SPY는 PreparedStatement의 `setString()` 호출 시점 — 즉 Oracle 서버에 값이 전달되기 **이전** 에 파라미터를 가로채 로깅한다.

  

```

[P6SPY 로그]

select LANG_CD, GROUP_CD, GROUP_NAME from T_CODEGROUP

where LANG_CD = 'KOR' and GROUP_CD = '002'

```

  

로그 상으로는 `'002'`가 올바르게 바인딩된 것처럼 보이지만, Oracle 내부에서는 이 값을 **VARCHAR2 타입**으로 처리하여 `'002  '`(CHAR)와 비교하므로 0건이 반환된다.

  

```

P6SPY 로그:  GROUP_CD = '002'    ← 눈에는 정상

Oracle 내부:  '002' (VARCHAR2) ≠ '002  ' (CHAR)  ← 실제로는 불일치

```

  

### 3.2 SQL Developer에서는 왜 되었나

  

SQL Developer에서 쿼리를 직접 실행할 때는 리터럴 문자열 `'002'`가 Oracle 내부에서 **CHAR 타입**으로 처리되어 공백 패딩 비교 규칙이 적용되므로 정상 조회된다.

  

| 실행 환경                  | 파라미터 타입  | 비교 규칙    | 결과    |     |
| ---------------------- | -------- | -------- | ----- | --- |
| SQL Developer (리터럴)    | CHAR     | 공백 패딩 비교 | 정상 조회 |     |
| JDBC PreparedStatement | VARCHAR2 | 비패딩 비교   | 0건 반환 |     |

  

이 차이가 "쿼리는 정상인데 애플리케이션에서만 안 된다"는 혼란을 유발했다.

  

---

  

## 4. 적용한 수정 내역

  

### 4.1 수정 전 (문제 코드)

  

```xml

<select id="selectGroupOne" resultMap="groupRm">

    SELECT LANG_CD, GROUP_CD, GROUP_NAME

    FROM T_CODEGROUP

    WHERE LANG_CD = #{langCd} AND GROUP_CD = #{groupCd}

</select>

```

  

### 4.2 수정 후 (수정 코드)

  

CHAR 컬럼에 `RTRIM`을 적용하여 공백 패딩을 제거한 후 비교하고, SELECT 시에도 정규화된 값을 반환한다.

  

```xml

<select id="selectGroupOne" resultType="kr.co.nh.atms.web.code.entity.CodeGroup">

    SELECT RTRIM(LANG_CD)  AS langCd,

           RTRIM(GROUP_CD) AS groupCd,

           GROUP_NAME      AS groupName

    FROM T_CODEGROUP

    WHERE RTRIM(LANG_CD) = #{langCd} AND RTRIM(GROUP_CD) = #{groupCd}

</select>

```

  

### 4.3 동일 문제가 적용된 전체 범위

  

`T_CODEGROUP`과 `T_CODE`의 CHAR 컬럼(`LANG_CD`, `GROUP_CD`, `CODE`)을 사용하는 **모든 WHERE 절과 SELECT** 에 일괄 적용하였다.

  
| 쿼리 ID               | 적용 내용                      |
| ------------------- | -------------------------- |
| `codeCols` (공통 컬럼)  | `RTRIM` + `AS` 별칭 추가       |
| `codeWhere` (공통 조건) | `RTRIM(컬럼) = #{파라미터}`      |
| `selectCodeOne`     | WHERE 절 RTRIM 적용           |
| `selectCodeByGroup` | WHERE 절 RTRIM 적용           |
| `updateCode`        | WHERE 절 RTRIM 적용           |
| `deleteCode`        | WHERE 절 RTRIM 적용           |
| `selectGroupList`   | SELECT RTRIM 적용 (반환값 정규화)  |
| `selectGroupOne`    | SELECT + WHERE 모두 RTRIM 적용 |
| `updateGroup`       | WHERE 절 RTRIM 적용           |
| `deleteGroup`       | WHERE 절 RTRIM 적용           |
| `countCodesByGroup` | WHERE 절 RTRIM 적용           |

  

> **INSERT는 수정 불필요**: INSERT 시 Oracle이 자동으로 CHAR 길이에 맞게 패딩하므로 정상 동작한다.

  

---

  

## 5. 재발 방지 대책

  

### 5.1 DB 스키마 설계 원칙 (신규 테이블)

  

**`CHAR` 타입은 사용하지 않는다. 고정 길이 코드성 컬럼도 `VARCHAR2`로 선언한다.**

  

```sql

-- ❌ 금지

GROUP_CD  CHAR(5)

  

-- ✅ 권장

GROUP_CD  VARCHAR2(5)  NOT NULL

```

  

`CHAR`는 비교 시 타입에 따라 동작이 달라지는 숨겨진 위험이 있다. Oracle에서 `VARCHAR2`는 입력 길이 그대로 저장하며 JDBC와 완전히 호환된다.

  

### 5.2 기존 CHAR 컬럼을 다루는 MyBatis 작성 규칙

  

레거시 DB의 `CHAR` 컬럼을 피할 수 없는 경우, 아래 규칙을 반드시 따른다.

  

```xml

<!-- ❌ 금지: CHAR 컬럼을 파라미터와 직접 비교 -->

WHERE GROUP_CD = #{groupCd}

  

<!-- ✅ 필수: RTRIM으로 공백 제거 후 비교 -->

WHERE RTRIM(GROUP_CD) = #{groupCd}

```

  

```xml

<!-- ❌ 금지: CHAR 컬럼을 그대로 SELECT -->

SELECT GROUP_CD FROM T_CODEGROUP

  

<!-- ✅ 권장: RTRIM으로 정규화하여 SELECT -->

SELECT RTRIM(GROUP_CD) AS GROUP_CD FROM T_CODEGROUP

```

  

### 5.3 코드 리뷰 체크리스트 항목 추가

  

PR 리뷰 시 아래 항목을 점검한다.

  

```

[ ] Oracle CHAR 타입 컬럼과의 WHERE 비교에 RTRIM이 적용되어 있는가?

[ ] CHAR 타입 컬럼이 SELECT에 포함된 경우 RTRIM 정규화가 되어 있는가?

[ ] 신규 테이블 DDL에 CHAR 타입이 사용되지 않았는가?

```

  

### 5.4 SQL Developer와 JDBC 동작 차이 인지

  

테스트 시 다음 사항을 반드시 확인한다.

  

| 점검 항목                            | 방법                                           |
| -------------------------------- | -------------------------------------------- |
| P6SPY 로그만으로 결과를 판단하지 않는다         | P6SPY는 바인딩 값을 보여주지만, 타입 정보는 표시하지 않는다         |
| SQL Developer 직접 실행 결과를 맹신하지 않는다 | 리터럴 문자열은 CHAR 비교, JDBC는 VARCHAR2 비교로 동작이 다르다 |
| 단건 조회는 목록 조회와 별도로 반드시 테스트한다      | WHERE 절이 없는 목록과 달리, 단건은 CHAR 비교 문제가 노출된다     |

  

### 5.5 디버깅 시 파라미터 길이 출력 패턴

  

CHAR 타입 문제는 값이 같아 보여도 길이가 다르면 발생한다. 의심 시 길이를 함께 로깅한다.

  

```java

log.debug("groupCd='{}' len={}", groupCd, groupCd == null ? -1 : groupCd.length());

```

  

DB 실제값과 수신 파라미터의 `len`이 다르면 CHAR 패딩 문제를 의심한다.

  

---

  

## 6. Oracle CHAR vs VARCHAR2 핵심 정리

  

| 항목                   | `CHAR(n)`                   | `VARCHAR2(n)`         |
| -------------------- | --------------------------- | --------------------- |
| 저장 방식                | 고정 길이 (공백 패딩)               | 가변 길이 (입력값 그대로)       |
| `= CHAR` 비교          | 공백 패딩 비교 (blank-padded)     | —                     |
| `= VARCHAR2` 비교      | **비패딩 비교 (non-padded)**     | 비패딩 비교                |
| JDBC String 비교       | **비패딩** → `'002' ≠ '002  '` | 비패딩 → `'002' = '002'` |
| SQL Developer 리터럴 비교 | 공백 패딩 → `'002' = '002  '`   | —                     |
| 해결 방법                | `RTRIM(컬럼) = #{param}`      | 별도 처리 불필요             |

  

> **핵심**: SQL Developer에서는 되고 애플리케이션에서는 안 된다면 CHAR vs VARCHAR2 비교 규칙 차이를 가장 먼저 의심하라.

  