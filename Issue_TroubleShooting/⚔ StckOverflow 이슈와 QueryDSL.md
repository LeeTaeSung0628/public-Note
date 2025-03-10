# ⚔ StckOverflow 이슈와 QueryDSL

#Spring #StckOverflow #QueryDSL #JPA #DB

---

- # 정말 오랜만에 보는 `StckOverflow`
![[Pasted image 20250307172522.png]]

---
# 원인 분석 전 사전지식

- Hello에서는 특별한 상황이 아닌 이상, **JPA - QueryDSL** 방식으로 코드를 통일하고 있다.
-> *QueryDSL: Type-Safe) 동적 SQL을 작성할 수 있도록 도와주는 Java 기반의 ORM(Query Builder)*
*가독성이 뛰어나며, 컴파일 시점에 오류 검출 가능*

- `BooleanExpression`은 **QueryDSL**에서 제공하는 **동적 쿼리 조합 기능**이다.
ex)
``` java
import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import java.util.List;

public List<User> findUsersByConditions(String name, Integer age) {
    QUser user = QUser.user;
    JPAQueryFactory queryFactory = new JPAQueryFactory(entityManager);

    BooleanExpression predicate = user.isNotNull();  // 기본 조건 (항상 참)

    if (name != null) {
        predicate = predicate.and(user.name.eq(name));
    }
    if (age != null) {
        predicate = predicate.and(user.age.gt(age));
    }

    return queryFactory.selectFrom(user)
            .where(predicate)
            .fetch();
}

```
### `predicate.and(condition)` 또는 `predicate.or(condition)`을 호출하면, 
**새로운 BooleanExpression 객체를 반환**한다.

## BooleanExpression이 사용된 이유
1. **함수형 조합** : 함수처럼 동적 쿼리 조합을 관리 할 수 있다.(여러 개의 조건을 동적으로 조합 가능)
2. **스레드 안정성** : 불변객체로 생성되므로, 여러 스레드에서 동시에 사용하더라도 안전하다.

---

# StckOverflow 발생 이유?

```java
BooleanExpression expr1 = user.name.like("A%");
BooleanExpression expr2 = expr1.and(user.age.gt(20));
```
BooleanExpression은 위에서 설명한 바와 같이, Immutable(불변)객체이다.
이 때, 위와같이 and/or 연상을 호출하면, **기존 객체를 변경하는 것이 아닌, 새로운 객체를 만들어 낸다.**

해당 연산은 연쇄적으로 새로운 객체를 생성하며, 깊이가 n이 되는 트리(Tree)구조가 형성된다.
*`or()` / `and()` 연산을 할 때마다 기존 객체를 참조하는 새로운 객체가 생성됨.*

### 이때, JVM의 한정된 스텍의 크기를 초과하는 스텍(Tree)구조가 쌓이게 된다면,
*StckOverflow*가 발생하게 되는 것이다.

---

# 해결방법?

## 방법 1. `BooleanBuilder` 활용 (QueryDSL 제공)

```java
import com.querydsl.core.BooleanBuilder;
import com.querydsl.core.types.dsl.BooleanExpression;
import java.util.List;

public BooleanExpression buildPredicateEfficiently(List<String> patterns) {
    QUser user = QUser.user;
    BooleanBuilder builder = new BooleanBuilder(); // 동적 조건을 쌓는 도구

    for (String pattern : patterns) {
        builder.or(user.name.like(pattern + "%")); // 재귀 호출 없이 추가
    }

    return builder;
}
```
- `BooleanBuilder`는 객체를 생성할 때, 내부적으로 BooleanExpression을 `List` 형태로 관리하게 된다.
- 따라서, 불필요한 재귀 호출 없이 메모리를 효율적으로 사용할 수 있다.
### 비교 분석

| 비교 항목        | `BooleanExpression.or()` 연속 사용 | `BooleanBuilder` 활용 |
| ------------ | ------------------------------ | ------------------- |
| **객체 생성 방식** | `Immutable(불변) 객체` 매번 새로 생성    | 내부적으로 리스트 관리        |
| **메모리 사용량**  | 매우 많음 (새로운 객체 계속 생성)           | 상대적으로 적음            |
| **재귀 깊이**    | 깊은 트리 구조 (StackOverflow 발생 가능) | 리스트 구조 (재귀 X)       |
| **성능**       | OR 조건이 많을수록 느림                 | 상대적으로 빠름            |
| **권장 여부**    | ❌ 비효율적                         | ✅ 추천                |


---

# 방법 2. `WHERE IN` 사용

```java
List<String> names = List.of("A", "B", "C");

List<User> users = queryFactory
    .selectFrom(user)
    .where(user.name.in(names))
    .fetch();

```
- 리스트(Set, List, 배열) 데이터를 한 번에 비교
- B-tree 인덱스를 활용하여 더 효율적인 검색 가능
- 대량 데이터를 비교할 때 성능이 더 좋음
### BooleanBuilder vs WHERE IN 성능 비교

|비교 항목|`BooleanBuilder (OR)`|`WHERE IN`|
|---|---|---|
|**SQL 변환 형태**|`WHERE name = 'A' OR name = 'B' OR name = 'C'`|`WHERE name IN ('A', 'B', 'C')`|
|**실행 계획 (EXPLAIN)**|여러 개의 `OR` 조건을 평가해야 하므로 느릴 수 있음|**`IN`은 단일 조건으로 평가되므로 더 빠름**|
|**인덱스 활용 가능성**|`OR` 연산이 많아지면 **인덱스가 제대로 활용되지 않을 가능성 높음**|**B-tree 인덱스를 활용하여 성능이 더 좋을 가능성이 큼**|
|**데이터 크기 증가 시 성능**|**조건이 많아지면 급격히 성능 저하**|**조건이 많아도 상대적으로 안정적**|
|**추천 사용 케이스**|**다양한 필드에 동적 조건을 추가할 때**|**단순한 리스트 데이터를 비교할 때**|
하지만!
1. WHERE IN절은 값의 일치(Equality Check, `=`)만을 지원하기 때문에, `Like`연산과 같은 패턴 매칭을 이용하기 위해선 *BooleanBuilder*를 사용해야한다.
2. 다중 컬럼 비교가 불가능하기 떄문에, 2개 이상의 컬럼을 비교하려면 BooleanBuilder를 사용해야한다.

---

# 결론
#### - 쿼리를 작성할때, 해당 쿼리의 목적을 정확히 파악하고 최선의 성능과 안정성을 고려하려 설계하도록,
#### 항상 고민하자.
