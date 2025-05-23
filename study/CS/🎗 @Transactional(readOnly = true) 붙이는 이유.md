# 🎗 @Transactional(readOnly = true) 붙이는 이유

#공부 #DB #Transaction #readOnly #JPA

---

# <font color="#76923c">개요</font>

오늘은 `@Transaction`, 특히 readOnly = true 옵션에 대하여 깊이 탐구해 보겠다. <br>
우리는 `@Transaction` 어노테이션을 통해(Spring AOP로) 트렌젝션 범위를 쉽게 구현할 수 있다.

여기서 조회만을 위한 로직에는 `@Transactional(readOnly = true)`옵션을 추가하여 *성능을 높여주는 사실*은 모두 알고있을 것이다. 여기서 <u>어떠한 로직을 거처 성능상의 이점</u>을 얻어내는지 탐구해 보겠다.

<br>

---

<br>

# <font color="#76923c">@Transactional 자세히 보기</font>

<br>

## 1. 트렌젝션이란 무엇인가?
#### 뜻
데이터베이스의 상태를 변경하기위해 수행하는 작업의 단위.

#### 목적
데이터의 무결성을 보장하는 것.

- **무결성** : 데이터의 정확성, 일관성, 유효성이 유지되는 것. <font color="#76923c">데이터 값이 정확한 상태.</font>
	- *정확성* : 중복이나 누락이 없는 상태.
	- *일관성* : 작업 전/후 데이터 상태가 일관되게 유지되는 상태.
	- *유효성* : 사용자로부터 값을 입력받을 때, 정확한 값만을 입력받도록 하는 것.

>[!caution] 데이터 무결성과 데이터 정합성의 차이
>
> 무결성은 위에서 설명한대로 데이터가 정확하고 유효한 상태고,
> 
> 정합성은 데이터의 값이 서로 일치하는 상태를 뜻한다.(반정규화를 통해 이상현상이 발생하면 정합성이 깨진다.)
> 
> 정합성은 지키지만 무결성이 깨진 상태의 예를 들자면,
> `주문테이블`과 `고객테이블`의 idx값이 모두 -1 이라면 정합성은 지키고 있지만, 반드시 1이상의 값을 가져야한다는 idx값의 규칙에 의해 **데이터 무결성이 훼손되게 된다.**


---

<br>

## 2. `@Transactional`의 기본 개념

<br>

- Spring에서 트렌젝션을 선언적으로 관리하기 위해 사용.
- 메서드 또는 클래스 단위로 트렌젝션 범위 지정.

사용시 다음과 같이 사용된다.
```java
@Transactional
public void doBusinessLogic() {
    ...
}
```

### <span style="background:#ff4d4f">주의점</span>

###  `@Transactional`의 위치와 프록시 메커니즘

- `@Transactional`은 Spring AOP의 **인터페이스 기반 (JDK 동적 프록시)** 또는 **클래스 기반 (CGLIB)** 으로 프록시를 생성한다.
- 따라서 내부 호출(self-invocation) 시 트랜잭션이 적용되지 않는다.
```java
public class A {
    @Transactional
    public void methodA() {
        methodB(); // 트랜잭션 적용 안됨!
    }

    @Transactional
    public void methodB() { ... }
}
```

## SpringAOP 사용 예 보러가기
#### ▶ [[👩‍👧‍👦 marketing Analytics 공통모듈 제작기]]


---

<br>

# <font color="#76923c">readOnly 옵션</font>

<br>

#### 목적
- 쓰기 작업 금지 : flush발생 X / 변경감지X
- 성능 최적화 : 특정 DB에서 Lock 생략 ▶ [[Lock ( 데이터베이스 락 ) 이란]]
```java
@Transactional(readOnly = true)
```
> 단순 조회 전용 메서드에 적용하여 리소스를 절약한다.

<br>

## 상세한 설명

- 위에서 설명한 바와 같이, 읽기 목적의 로직에 여러 작업을 간소화 하여 성능을 높이는 옵션이다.
- 이를 더 상세히 설명하겠다.

<br>

## 1. 영속성 컨텍스트와 변경감지

▶ [[영속성 컨텍스트]]

#### 변경감지(Dirty Checking)
- 영속성 컨텍스트는 Entity조회 시 초기 상태에 대한 SnapShot을 저장한다.
- <u>트렌젝션이 Commit</u> 될 때, *초기 상태(SnapShot)* 와 *현재 Entity의 상태*를 비교한다.
- 이때, 비교된 내용에 대해 update query를 자동으로 생성해 쓰기 지연 저장소에 저장한다.
- 이후 일괄적으로 query를 `flush()`하여 DB에 반영되게 된다.

이와같이 사용자가 직접 update쿼리 등을 작성하지 않아도, **엔티티의 변경점을 감지하여 자동으로 수정/반영 해주는 것이 영속성 컨텍스트의 변경감지 기법이다.**

<br>

위서 말했듯, `readOnly = true` 옵션이 켜져있다면, flush발생과 자동감지의 동작을 멈춘다.
즉, JPA세션의 플러시 모드를 MANUAL로 변경하여 <u>강제로 flush를 호출하지 않는 한, 엔티티 수정내역에 DB에 반영되지 않는다.</u>


#### 결론
- `readOnly = true`옵션이 켜져있다면, JPA는 해당 트렌젝션이 내의 조회 쿼리에 대해서, 변경감지를 위한 초기상태(SnapShot)을 저장하지 않아도 되고, 불필요한 Dirty Checking을 건너 뛰게 함으로 **매모리와 CPU리소스를 절감할 수 있다.**

<br>

## 2. 읽기전용 코드에 대한 가독성 증가

```java
// 1
@Transactional(readOnly = true)
public Member getMember(int memberId) {
    ..
    return member;
}

// 2
@Transactional
public Loaner getLoaner(int memberId) {
    Optional<Loaner> loaner = loanerRepository.findById(loanerId);
    ..
    return loaner;
}
```

- 위 코드를 보았을 때, 물론 코드를 분석하여 단순 select로직인지, 쓰기 로직인지 판단할 수있겠지만, <br>
**누가 봐도 단번에 `getMember(int memberId)`가 읽기전용 메서드라는것을 파악할 수 있다.**
- 이는 코드의 가독성을 향상시켜 줄 뿐 아니라, 실수로 트리거 할 수 있는 쓰기 작업을 거부할 수 있다.
<br>

## 3. 레플리케이션(Replication) 환경의 분산 부하

>[!tip] 레플리케이션이란?
>’두개 이상의 DBMS를 Master/Slave 라는 수직적 구조를 활용하여 DB의 부하를 분산시키는 기술’
>
>Master DB에는 insert/update/delete 와 같은 작업을 수행하도록 하고, select 작업을 Slave DB에서
>
>작업하도록 구성한다.

<br>

#### 왜? select작업만을 따로 빼는 구조를 만들었을까?

- 보통의 경우 select작업의 소요시간이 가장 길기 때문이다.
- Table Full Scan에 경우 데이터 개수에 따라 소요시간이 아주 길게 사용될 수 있다.
- 이 시간동안, 다른 작업을 하지 못하게 되니 병목이 발생하는 주요원인이 된다.


#### 이로 인한 장점
1. 레플리케이션 구조는 복제본 DB(Slave)를 함께 운용함으로, Master DB장애 발생시 SlaveDB를 승격시켜 **장애를 빠르게 복구**할 수 있다.
2. 조회작업에 대한 트레픽을 분산할 수 있다.

![[do-messenger_screenshot_2025-05-23_10_48_34.png|725]]

## 결론

- @Transactional(readOnly = true) 옵션은 SlaveDB에서 데이터를 가져오도록 동작시키고,
	- (읽기 전용 트렌젝션이 SlaveDB로 자동 라우팅 된다.)
- 이를 통해 **레플리케이션**의 목적에 맞게 트래픽 분산을 온전하게 적용시킬 수 있다.

<br>

---

<br>

# + <font color="#76923c">@Transactional 어노테이션을 제거한다면?</font>

<br>

만약, readOnly=ture 옵션을 통해, 스냅샷 저장을 막고 조회 속도를 올리는게 목적이라면,

@Transactional 어노테이션을 완전히 지우면 되는것 아닌가?

```java
// @Transactional(readOnly = true) -> 주석처리
public Member getMember(int idx) {
        Member member = memberRepository.findByIdx(idx).get();
        System.out.println(member.getName()); // Lazy Loading 발생
        return member;
}
```

다음과 같이 코드를 구성했다면?
- 사실 아무일도 일어나지 않는다. (Lazy Loading이 정상적으로 동작한다.)
- **단, OSIV 옵션이 true인 경우에만 한정이다.** ▶ [[🌋 OSIV란 무엇인가]]

<br>

그렇다면 다음과 같이 osiv옵션을 false로 하고 재실행한다면?
```java
// application.properties
spring.jpa.open-in-view=false
```


`LazyInitializationException`이 발생하게 된다.

이는 영속성 컨텍스트가 종료된 이후(준영속 상태)에 해당 엔티티에 접근하려 할 때 발생하는 예외이다.

(<u>DTO로 변환하여 직접 엔티티에 접근할 필요가 없도록 관리하는 이유이기도 함</u>)

<br>

따라서, OSIV옵션이 켜져있지 않다면, @Transactional 어노테이션은 강제되고,

조회용 쿼리만을 위한 메서드에는 항상 readOnly=ture 옵션을 붙여주는것이 바람직하다.