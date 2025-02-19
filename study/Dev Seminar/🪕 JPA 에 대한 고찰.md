# 🪕 JPA 에 대한 고찰

#공부 #JPA #DB #SPRING

---

# JDBC - Java Database Connectivity

- java에서 DB를 사용할 수 있도록 연결해주는 응용프로그램 인터페이스인 java api이다.
- java와 연동되는 DBMS에 따라 그에 맞는 JDBC(ex. MySQL Connector)를 설치해야 한다.
- 그에 맞는 드라이버만 존재한다면, java에서 DB에 구애받지 않고 똑같은 코드로 사용이 가능하다.
#### 즉 JDBC는 공통으로 사용가능한 인터페이스이며, 각 DB에 맞는 구현체는 JDBC드라이버이다. - MySQL Connector

### 특징
- SQL명령을 명시적으로 작성해야 한다.
- 객치지향 프로그래밍과의 괴리가 있다.

----

# ODBC - Open Database Connectivity

- DB를 엑세스 하기 위한 ==표준 개방형 응용 프로그램 인터페이스==를 뜻한다.
- ODBC는 JDBC와 달리 모든 어떠한 DBMS인지와 상관없이 적용된다. 

---

# 그렇다면 JPA(Java Persistence API)는 무엇일까?

- jpa는 서로 다른 두 개념을 매핑해주는 ORM(Object Relational Mapping)중 하나이다.
- DB와 java객체간의 매핑을 자동화하여, SQL대신 java객체와 상호작용 하도록 설계되어있다.
	- RDB는 하나의 row를 하나의 인스턴스라고 생각한다면 컬럼의 값은 필드의 값으로 매핑할 수 있다.
	- 하지만 객체의 필드에 리스트가 존재하는 경우 상황이 애매해진다.
	- 또한 java객체의 필드에 또 다른 객체가 존재한는 경우 java는 참조를 하지만, DB는 Join으로 접근하기 때문에 다르다.
	- 이러한 비슷한 두 개념을 매핑해주는 역할을 하는것이다.

#### 그렇다면 JDBC와 JPA는 서로 다른것인가?

JPA는 DB와 객체를 매핑하는 기술일 뿐,
내부적으로 DB와 통신을 위해서는 JDBC를 필요로 하게된다.

또한 JPA도 JDBC와 마찬가지로 인터페이스이기 때문에 구현체가 필요하고, 그 구현체 중 하나가 ==Hibernate==이다.

---

# Spring Data JPA

- Spring Data JPA란 JPA를 Repository 기반으로 간편하고 효율적으로 사용할 수 있는 모듈이다.
- Repository의 메서드를 통해 쿼리를 날릴 수 있다.
- 또는 직접 쿼리를 만들고 싶다면 @Query 어노테이션을 사용한다.

---

# 대략적인 관계도
 ![[Pasted image 20250110144021.png]]

---

# JPQL(Java Persistence Query Language)는 또 무엇인가?

- JPQL은 JPA에서 사용되는 쿼리 언어로, SQL과 비슷하지만 객체를 대상을 쿼리를 작성하게된다.
- 테이블이 아닌 엔티티 객체를 대상으로 하기 때문에, 객체지향적이다.

### 특징
- SQL 유사성 : select, where, group by 등 SQL 과 유사한 문법 사용
- 엔티티 중심 : DB테이블 대신 엔티티 클래스와 속성을 기준으로 작성한다.
- 동적 쿼리 지원 : 런타임에 JPQL 문자열 생성 실행 가능
- 별칠(alias) 사용 필수

### 문제점
- 1. 문자열(String)형태 이기에 개발자 의존적 형태이다
- 2. 컴파일 단계에서 타입체크가 불가능하다
- 3. 런타임단계에서 오류가 발생할 수 있다

---

# Query DSL(Domain Specific Language)???

- 위에서 기술한 JPQL의 문제점을 보완하기 위해 나온것이 query dsl이다.
- 정적 타입을 이용해, SQL, JPQL을 코드로 작성할 수 있도록 도와주는 오픈소스 API

### 특징
- 컴파일 단계에서 오류 확인 가능, 후속 조치 가능 
- JPQL의 단점들을 거의 보완

### 단점
- 1. 코드가 너무 길어진다... 
- 2. FROM절의 서브쿼리를 사용하는것에 제약이 있다.
- 3. 세세한 튜닝이나, DBMS의 고유기능을 유연하게 사용할 수 없다.

---

# JPA n+1 문제? 

#### **1. 개념**

N+1 문제는 데이터베이스와 애플리케이션 간의 비효율적인 쿼리 실행으로 인해 발생하는 성능 문제를 지칭합니다.  
이는 주로 **ORM(Object-Relational Mapping)** 기술(JPA, Hibernate 등)을 사용할 때 발생하며, 한 번의 데이터베이스 조회로 해결할 수 있는 작업에 대해 **추가적인 N개의 쿼리가 실행되는 상황**을 의미합니다.

#### **2. 어떻게 발생하나?**

N+1 문제는 주로 **지연 로딩(Lazy Loading)**으로 인해 발생합니다.  
지연 로딩은 관련 엔티티를 필요할 때만 로딩하는 방식으로, 기본적으로 효율적인 방법이지만 아래와 같은 상황에서 문제가 발생합니다.

##### **예시 상황**

1. **두 개의 엔티티 간 관계**:
    
    - Parent(부모 엔티티)와 Child(자식 엔티티) 관계.
    - Parent 1개에는 여러 Child가 연결.
2. **문제 발생 과정**:
    
    - 부모 엔티티를 조회하는 쿼리 1번 실행.
    - 각 부모 엔티티에 연결된 자식 엔티티를 조회하는 쿼리 N번 실행.

``` java
List<Parent> parents = entityManager.createQuery("SELECT p FROM Parent p", Parent.class).getResultList();

// 각 Parent 엔티티의 자식 엔티티를 로드 (지연 로딩)
for (Parent parent : parents) {
    System.out.println(parent.getChildren().size()); // 자식 엔티티 조회 쿼리 발생
}

```

##### **쿼리 실행**

1. `SELECT * FROM Parent;` → 부모 엔티티를 조회하는 쿼리 1번 실행.
2. `SELECT * FROM Child WHERE parent_id = ?;` → 각 부모 엔티티마다 자식 데이터를 조회하는 쿼리 N번 실행.

- 결과적으로 **1 + N개의 쿼리**가 실행됩니다.
- 데이터가 많을수록 성능 저하가 극심해집니다.

---

#### **3. N+1 문제의 영향**

- 데이터베이스와 애플리케이션 간의 **불필요한 트래픽 증가**.
- 쿼리 실행 횟수가 많아져 **응답 시간이 느려짐**.
- 대량의 데이터가 있을 경우 애플리케이션 성능이 크게 저하.

#### **4. 해결 방법**

##### **(1) 페치 조인(Fetch Join)**

- 페치 조인은 SQL의 `JOIN`을 사용해 관련된 엔티티를 한 번의 쿼리로 로드하는 방식입니다.
``` java
String jpql = "SELECT p FROM Parent p JOIN FETCH p.children";
List<Parent> parents = entityManager.createQuery(jpql, Parent.class).getResultList();
```
- `JOIN FETCH`를 사용하면 부모와 자식 엔티티를 한 번에 가져옵니다.
- **장점**: 데이터베이스 쿼리를 최소화.
- **단점**: 데이터가 많을 경우 메모리 사용량 증가.

##### **(2) 엔티티 그래프(Entity Graph)**

- JPA에서 제공하는 기능으로, 로딩 시 어떤 관계를 함께 로드할지 명시적으로 정의합니다
``` java
@Entity
@NamedEntityGraph(
    name = "Parent.withChildren",
    attributeNodes = @NamedAttributeNode("children")
)
public class Parent { ... }

// 사용 시
EntityGraph<?> entityGraph = em.getEntityGraph("Parent.withChildren");
List<Parent> parents = em.createQuery("SELECT p FROM Parent p", Parent.class)
    .setHint("javax.persistence.fetchgraph", entityGraph)
    .getResultList();

```
- **장점**: 명시적으로 필요한 관계만 가져올 수 있음.
- **단점**: 추가적인 설정 필요.

##### **(4) DTO로 데이터 조회**

- 필요한 데이터만 직접 SQL 또는 JPQL로 조회하여 반환.
``` java
String jpql = "SELECT new com.example.dto.ParentChildDTO(p.name, c.name) " +
              "FROM Parent p JOIN p.children c";
List<ParentChildDTO> dtos = entityManager.createQuery(jpql, ParentChildDTO.class).getResultList();

```
- **장점**: 불필요한 엔티티 로드 방지.
- **단점**: 객체 매핑 작업 추가.
---

# 트렌젝션 / DB스냅샷 / 앤티티매니저 / 영속성컨텍스트 / JPA

## 각각의 정의

#### 트렌젝션
	- 데이터베이스의 상태를 변화시키기 위해 수행하는 작업의 단위

#### 트렌젝션매니저
	- Spring에서 제공하는 트렌젝션 관리 기능과 JPA를 연결하는 역할을 수행.

#### @Transactional
	- 클래스나 메서드에 삽입하면, AOP레벨(프록시를 사용하여)에서 트렌젝션매니저를 이용한 동작을 공통으로 적용.

#### 앤티티매니저
	- 영속성 컨텍스트를 관리하는 인터페이스
	- 엔티티의 저장/수정/삭제/조회 작업을 수행
	- 스레드 세이프하지 않으므로 한트레젝션 내에서만 사용
	- 앤티티매니저펙토리는 스레드세이프하므로 공유 가능

#### 영속성컨텍스트
	- JPA의 엔티티를 관리하는 1차 캐시 역할을 하는 메모리 공간
	- 엔티티와 DB데이터간의 상태 동기화를 책임
	- 1차캐싱 / 변경 감지 / 지연 로딩 의 특징을 갖는다
	- 지연로딩 : 관계된 엔티티를 실제로 필요할때만 가져오며, 변경점을 한번만 commit한다

#### 스냅샷
	- DB스냅샷 : 
		- 트렌젝션이 시작될때 생성된다.
		- 독립적인 데이터베이스 복사본으로, 원본데이터 변경과 무관하다.
		- 고급 격리 수준(REPEATABLE READ, SERIALIZABLE)에서 MVCC(Multi-Version Concurrency Control)를 구현할 때 사용한다.
		- 물리적인 복사본이 아닌, 논리적으로 매 쿼리마다 동적으로 가공이 되는 방식이다.
	- 앤티티매니저_스냅샷 : 
		- 엔티티가 영속성 컨텍스트에 로드될 때 생성된다.
		- 엔티티의 초기 상태를 저장하여 변경 감지에 사용된다.
