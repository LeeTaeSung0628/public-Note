# ⚔ StckOverflow 이슈와 QueryDSL

#Spring #StckOverflow #QueryDSL #JPA #DB

---

- # 정말 오랜만에 보는 `StckOverflow`
![[Pasted image 20250307172522.png]]

---
# 원인 분석 전 사전지식

- Hello에서는 특별한 상황이 아닌 이상, **JPA - QueryDSL** 방식으로 코드를 통일하고 있다.
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
- 

---

# StckOverflow 발생 이유?

