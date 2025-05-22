# 🍧 JPA의 영속성 상태와 데이터 Log 이슈

#SQL #JPA #이슈 #log #로그

---

<br>

# <u><font color="#76923c">개요</font></u>

업무 중 한 가지 **이슈**가 있었고, <u>JPA의 영속상태와 동작원리</u>에 대해서 자세히 짚고 넘어가야할 필요가 있다고 느껴
해당 글을 쓰게 되었다.

상황은 다음과 같다.

> 1. 신한은행의 계좌잔액(**예치금**)과 입금내역을 관리하는 DB의 계좌 잔액간의 **차액**이 발생했다는 알림을 받았다.
> ### [[🏹 예치금 차액 비교 Spring Batch 리펙토링]]
> 2. 원인을 찾기위해 <u>*신한 전문을 쌓는 log테이블*</u>과, <u>*회원 입급내역 log테이블*</u>을 비교하였다.
> 3. 이떄, 신한DB 에는 같은 금액의 중복 log가 없으나, **입금 내역 DB에는 차액만큼의 중복 입금 log**를 발견할 수 있었다.

---

<br>

# <u><font color="#76923c">분석</font></u>

<br>

## 원인이 무엇일까?

차액이 **138만원** 발생했다고 가정했을 때,
1. 입금내역DB에는 10시 / 11시 총 *2개의 138만원 입금내역이* LOG로 남아있었다.
	- 여기서 해당 입금내역이 차액을 발생시켰을 것으로 예상할 수 있었다.

<br>

2. 실제 신한의 전문을 저장하는 DB에는 *11시의 입금내역만이 존재하고 있었다*.

<br>

3. 신한 측문의 결과 입금에 문제가 생겼을 경우 해당 입금 전문을 *동일하게 한 번 더 보낸*다는 사실을 알 수 있었다. 

#### 다음 상황으로 미루어 보았을 때, 첫 번째(10시 입금 전문)이 사졌다는 것을 볼 수 있다.

<br>

## 왜 사라졌을까?

보통의 상황이라면, 동일한(id값 동일) 전문을 받아 DB에 insert된다면 SQL Exception이 터졌을 것이라 생각하였다.
하지만 상황으로 미루어 볼때, **insert가 아닌 update가 동작했을 것으로 예상할 수 있다**.

1. 해당 Insert쿼리는 Spring DATA JPA의 *SAVE메서드로 구현*되어있다.
2. 첫번째 save 후 두 번째 save동작 까지, **1시간의 시간 차이**가 있다.

트렌젝션이 종료된 이후(commit)임에도 기존의 id를 기억하여 update를 할 수 있었던 이유가 무었일까?

#### 이에 대한 해답을 알기위해 *영속 상태*의 개념에 대해 알아보겠다.

---

<br>

# <u><font color="#76923c">영속 상태</font></u>

영속상태에 관한 관련된 또 다른 이슈.
## ▶ [[🍂 JPA, Mybatis , Dead Lock이슈]]

<br>

## 영속성 컨텍스트와 영속 상태란?

영속성 컨텍스트에 대한 설명.
## ▶ [[영속성 컨텍스트]] 


![[Pasted image 20250519144842.png|700]]

#### 나의 예상대로 라면...
- 아래 코드의 실행 결과로 알 수 있듯이, <u>**비영속 상태의 엔티티**를 `save()`하여 `persist()`를 수행했을 때</u>, DB에 이미 동일한 ID(PK)가 있다면 **예외를 발생시킨다**.
```java
@Autowired  
private EntityManager em;  
  
@Test  
@Transactional  
void MemberPersistenceTest() {  
  
    // 1) 새 엔티티 인스턴스 생성 → Transient 상태  
    HfMarketingCode testcode = new HfMarketingCode();  
  
    testcode.setHitCode("testCode1");  
    testcode.setCodeName("testName1");  
  
    em.persist(testcode);  
  
    em.flush();  
}
```
![[Pasted image 20250520142253.png]]
>[!danger] 주의
> 영속성 컨텍스트에 등록할 객체의 id 설정의 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 
> 여부에 따라 주의해야 할 사항이 있다.
> 1. *지연 쓰기*가 동작하지 않을 수 있다. 기본키 생성에 대한 권한을 DB에 위임하기 때문에, *JPA가 곧바로 id값을 알기위해 지연하지 않고, 바로 insert쿼리를 실행*시킨다(IDENTITY 방식의 경우).
> >
> 2. id값을 명시적으로 지정한 후, `persist()`를 수행하면 예외가 발생한다. 그 이유는, 명시적으로id를 지정하는 순간 non-null의 id값을 갖게되고, `isNew()`의 첫 호출부터 id가 null이 아니기 때문에, 기존에 존재하는 id값에 대해 persist를 수행하게 되어 예외가 발생하는 것이다.

아래는 JPA save()의 isNew() 분기문
```java
@Transactional  
public <S extends T> S save(S entity) {  
    Assert.notNull(entity, "Entity must not be null.");  
    if (this.entityInformation.isNew(entity)) {  
        this.em.persist(entity);  
        return entity;  
    } else {  
        return this.em.merge(entity);  
    }  
}
```
---

>[!question] 질문?
> 동일한 idx(pk)의 엔티티를 넘겨 save동작을 수행했을 때, *persist(insert)* 가 아닌 *merge(update)* 가 되었다면,
> 해당 엔티티의 영속 상태는 어떻게 되는가?

### 해답 :
> 엔티티 메니저는 트렌젝션이 종료될때 close되며, 이때 모든 영속석 **컨텍스트에 등록된 엔티티를 준영속 상태로 돌린다**.
> 따라서, 준영속 상태로 관리되고 있던 객체에 save() 연산이 수행되면서, update쿼리가 실행된 것.

<br>

그렇다면 준영속 상태의 지속 범위는 어떻게 될까?
- 보통 엔티티 객체가 준영속 상태로 진입하게 되면, *엔티티 매니저*와 모든 의존성을 끊기 때문에 일반적인 POJO 객체와 같이 *GC(가비지 컬렉터)* 에 의해 메모리를 해제하게 된다.
- 그럼에도 1시간의 시간 차가 발생했음에도 GC로 정리가 되지 않은 부분은 조금 의아하다.
- 해당 부분은 더 깊게 찾아보야 할것으로 보인다.

---

<br>

# + <font color="#c0504d">추가</font>
- 위에서 다음과 같이 표현한 부분이 있다. 이는 불가능하다는 것을 알게 되었다..
>*“ 비영속 상태의 엔티티를 `save()`하여 `persist()`를 수행했을 때,*
>
>*DB에 이미 동일한 ID(PK)가 있다면 예외를 발생시킨다 ”*


- 물론, 이미 존재하는 id값을 갖는 엔티티를 `persist()`하면 pk중복 예외가 방생하는 것은 맞다. `persist()`는 영속성 컨텍스트에 등록여부를 판단 할 뿐, id값의 유뮤를 따지지(select하지) 않기 때문이다.
- *하지만 이러한 상황이 발생하는 것이 <u>정상적인 상황에서 불가능하다.</u>*
- 그 이유에 대해서 설명하겠다.

#### 이유
: Assigned(사용자 id 직접 지정) 전략일 때, `id == null or 0` 일때만 “새 엔티티로써 판단한다.”

```java
public boolean isNew(T entity) {  
    ID id = this.getId(entity);  
    Class<ID> idType = this.getIdType();  
    if (!idType.isPrimitive()) {  
        return id == null;  //null 이거나,
    } else if (id instanceof Number) {  
        return ((Number)id).longValue() == 0L;   //0 일때만 새로운 객체로 판단
    } else {  
        throw new IllegalArgumentException(String.format("Unsupported primitive id type %s", idType));  
    }  
}
```
하지만 이때, Assigned으로 직접 id에 값을 어플리케이션에서 직접 지정했다면, (id값을 포함한 엔티티)
**`persist()`가 아닌 `merge()`로 넘어갈 수밖에없게 된다는 것이다.**

#### 즉, 어떠한 객체가 save() 되는 시점에 `persist()`가 동작되어 <u>PK중복 예외가 발생할 일은 없다는 것이다.</u>

<br>

---
<br>

## <font color="#76923c">결론</font>
: JPA의 save()는 단순히 insert와 update의 통합이 아니다. 각 동작의 원리와 특성을 파악하여 예외사항을 정확히 판단하여 결과를 예상 가능하도록 설계해야한다.