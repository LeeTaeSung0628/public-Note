# 🍂 JPA, Mybatis , Dead Lock이슈

#세미나 #이슈 #JPA #Java #Mybatis #DeadLock

---
>[!tip] 정보
> 해당 내용은 Hello 주간 세미나 중 주제로 선정된,
>  **투자하기 Dead Lock이슈 해결과정**에 대한 설명이다.
 
 ----
 
# **문제**

- server log
```
Caused by: org.apache.ibatis.exceptions.PersistenceException:
###Error querying database. Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
###The error occurred while executing a query
###Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
Caused by: java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
at com.zaxxer.hikari.pool.HikariPool.createTimeoutException(HikariPool.java:696)
at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:197)
at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:162)
```
해당 서버의 `maximum-pool-size`는 **40**으로 설정되어있으며,
**30**초의 대기를 했음에도 Connection Pool을 할당받지 못한 상황이다.


- 위 조건 발생 코드 예시
``` java
public void deadLockMethod(){
    mybatisSelectMethod();    
    
    jpaSelectMethod();
    
    mybatisSelectMethod();
}
```

---

# **원인 분석**

## 단순 부하 문제인가?
- 해당 로직은 몇개월 전 수정 된 이후, 계속해서 정상적으로 운영되던 코드이다.
- 순간적으로 트레픽이 몰린 상황을 가정하더라도, **40개의 pool이 30초간 점유를 지속한 것은 비정상 적**이다.

## deadlock이 원인인가??
- 일반적으로 deadlock은 **DB레벨**에서의 트렌젝션이 서로 기다리는 경우에 많이 발생한다.
- 하지만, Stack trace를 확인하였을 때, DB레벨의 deadlock은 아니었다.

## Connection Pool 상호 선점 후 대기?
- 서비스 로직에서 한 서비스가 커넥션풀을 반환하지 않고, 또 다른 서비스가 커넥션 풀을 요청하면 무한 순환이 발생할 가능성이 있다.

즉, **위 예시 코드**에서 Connection Pool을 반환하지 않고 무한정 대기할 가능성이 가장 크다고 판단하였다.

---

# **Connection Pool을 반환하지 않는 이유?**


```java
public void deadLockMethod(){
    mybatisSelectMethod();    
    
    jpaSelectMethod();
    
    mybatisSelectMethod();
}
```
커넥션 풀 상호 점유가 일어나고 있는 서비스 로직을 보았을 때, 특별한 점은 보이지 않는다.
그러나, 한가지 
**mybatis와 jpa를 혼용해서 사용중인 로직인 점이 눈에 뛴다.**

#### 일반적인 생각으로는?
*@Transactional* 이 걸려있지 않기 때문에, **순차적으로**

1. mybatisSelectMethod(); : 커넥션 풀 점유 후 반환
2. jpaSelectMethod(); : 앤티티 매니저에서 커넥션 풀 점유 후 close(반환)
3. mybatisSelectMethod(); : 커넥션 풀 점유 후 반환
<u>(이때 커넥션풀이 전부 점유중이라면 대기 30s )</u>

의 순서로 진행될 거라고 생각했다.

---

## MyBatis와 JPA 동작 로직 단순 비교

### MyBatis

MyBatis는 내부적으로 JDBC 커넥션을 관리하지 않고, DataSource를 통해 커넥션을 가져옴.
즉, **Spring에서 설정한 커넥션 풀(HikariCP, DBCP 등)**을 통해 커넥션을 관리한다.**

#### 기본 흐름
1. MyBatis가 DataSource(ex: HikariCP)에서 커넥션을 요청
2. SQL 실행 (SELECT 문 수행)
3. 커넥션이 자동 반환됨 (커밋/롤백 필요 없음)

### JPA

jpa또한 커넥션 풀을 통해 커넥션을 관리하지만, 영속성 컨텍스트(엔티티 매니저)에 권한을 위임한다.
*앤티티 매니자 : 영속성 컨텍스트를 관리하는 핵심 객체*
#### 기본흐름
1. jpa가 앤티티 매니저를 통해 DataSource(ex: HikariCP)에서 커넥션을 요청
2. SQL 실행 (find(SELECT 문) 수행)
3. 엔티티 매니저 Close 
4. 커넥션 반환
#### **비슷한 Connection Pool time out 이슈 보러가기**
### [[🚦 Spring Batch 트러블슈팅]]

---

## **어느 부분에서 문제가 발생했나?**

- 앤티티 매니저가 Connection Pool을 반환하는 시점은 언제인가

[영속성 컨텍스트의 지속 범위](https://velog.io/@seyoung755/%EC%82%BD%EC%A7%88%EA%B8%B0-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%9D%98-%EC%A7%80%EC%86%8D-%EB%B2%94%EC%9C%84-feat.-OSIV)
-> 서치 중 위 내용을 참고하여, **OSIV** 라는 것을 알게되었다.

### OSIV(Open Session In View)란?
#### 자세한 설명보러가기 ▶
## [[🌋 OSIV와 영속성 컨텍스트]]

![[Pasted image 20250314122906.png|675]]
OSIV란 View에 데이터를 전달할 때 지연 로딩 등의 이유로 영속성 컨텍스트를 지속해야 하는 경우에 사용되는 것이다.
**즉, 영속성 컨택스트(앤티티 매니저)의 생명주기를 웹 요청이 끝날 때 까지 연장하는 옵션이다.**
**이 설정은 어플리케이션에 별다른 설정을 하지 않았다면 default `ON` 상태이다.**


```java
public void deadLockMethod(){
    mybatisSelectMethod();  -- 1
    
    jpaSelectMethod(); -- 2
    
    mybatisSelectMethod(); -- 3
}
```
다시 한번 위 코드를 보자.

트래픽이 몰려 커넥션 풀 40개가 전부 점유되었을 때를 가정하자.

---

**조건**
- 여러개의 클라언트의 요청이 동시에 발생
- 2개 이상의 클라이언트가 *2번함수*를 수행 후, **영속성 컨택스트를 유지 중**
- *3번함수*를 실행하려고 하나, 커넥션 풀이 가득차 **대기상태에 돌입**

**실행**
1. 개발자는 *2번함수*는 동작을 완료한 후 커넥션풀이 해제되길 기대함.
2. OSIV 옵션이 켜져있을때, **Lazy Loading**이 view레이어 까지 이어짐.
	-즉, 동작이 완료되어도 **커넥션풀을 해제하지 않음**.
3. *3번함수*는 커넥션풀이 해제되길 무한정 기다림.

**<font color="#ff0000">정말 다음과 같이 동작할까???</font>**

>[!fail] 의문점
> OSIV가 활성화되어 있어도, 트랜잭션이 끝나면 Hibernate는 커넥션을 즉시 반환할 텐데,
> 어째서 무한 점유가 발생할 수 있는가?

---

# ~~**해결방법**~~

## ~~1. OSIV옵션 OFF~~
- ~~해당 서버의 옵션을 끄면, 데이터 일관성 문제 및 커넥션 점유 문제를 해결 가능~~
~~*한계 :* 해당 서버의 다른 서비스 까지 직접적인 영향을 끼침~~

## ~~2. @Transactional ?~~
- ~~@Transactional어노테이션은 서비스 레이어의 트렌젝션 관리에 쓰이는 것 이기 떄문에,~~
 ~~OSIV 옵션으로 인한 view레이어의 지연로딩을 막을 수 없음~~

---
### Reference

[JPA, Mybatis 동시 사용시 발생할 수 있는 HikariCP Dead lock 해결 여정 ( feat.OSIV )](https://velog.io/@12onetwo12/%EC%9E%A5%EC%95%A0%ED%9A%8C%EA%B3%A0-DBCP-Connection-Leak-%ED%95%B4%EA%B2%B0-%EC%97%AC%EC%A0%95-feat.HikariCP-Dead-lock-QueryDSL)


