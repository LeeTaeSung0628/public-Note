# JPQL과 QueryDSL 비교

## JPQL 
``` java
	String username = "java";
	String jpql = "select m from Member m where m.username = :username";
	List<Member> result = em.createQuery(query, Member.class).getResultList()
```

jpql이란 JPA의 일부로, 쿼리를 Table이 아닌 객체 기준으로 작성하는 객체지향 쿼리 언어 이다.

문제점 : String형태 이기 때문에 개발자 의존적인 형태를 띈다.
	컴파일 단계에서 Type-Check가 불가능하다.
	런타임 단계에서 오류가 발생한다.(장애 리스크가 증가한다)

-----------------

## QueryDSL
``` java
	String username = "java";
	
	List<Member> result = queryFactory
        .select(member)
        .from(member)
        .where(usernameEq(username))
        .fetch();
```

QueryDSL은 해당 문제를 해결하기 위해서 나온 기능이다.

장점 : 문자가 아닌 코드로 쿼리를 작성할 수 있어 컴파일 시점에 문법오류 확인 가능.
       IDE의 자동완성 기능의 도움을 받을 수 있다.
       복잡한 쿼리나 동적 쿼리 작성이 편하다.
       쿼리 작성시 제약조건 등 메서드를 추출해서 재사용할 수 있다.



---