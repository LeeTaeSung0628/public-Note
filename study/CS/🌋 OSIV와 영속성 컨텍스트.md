# 🌋 OSIV와 영속성 컨텍스트

#공부 #Java #JPA #OSIV #영속성컨텍스트

---

# OSIV(Open Session In View) 란?
- OSIV란 View에 데이터를 전달할 때 지연 로딩 등의 이유로 영속성 컨텍스트를 지속해야 하는 경우에 사용되는 옵션이다.

즉, 영속성 컨택스트(앤티티 매니저)의 생명주기를 웹 요청이 끝날 때 까지 연장하는 옵션이다.
**이 설정은 어플리케이션에 별다른 설정을 하지 않았다면 default `ON` 상태이다.**

서버 실행 시 아래와 같은 로그를 확인할 수 있다.
```java
spring.jpa.open-in-view is enabled by default. Therefore, database 
queries may be performed during view rendering.Explicitly configure 
spring.jpa.open-in-view to disable this warning
```

---

# OSIV의 동작원리

![[Pasted image 20250314152143.png]]

1. OpenSessionInViewFilter 초기화
	- OpenSessionInViewFilter는 Hibernate(JPA객체)세션을 **요청 전체 처리동안 열어두기 위한 서블릿 필터이다.**
	- 해당 필터는 sessionFactory의 openSession메서드를 호출하여 새로운 Hibernate세션을 얻는다.

2. 요청 처리 시작  
    - doFilter 메서드가 **FilterChain 객체에 의해 호출되어 요청의 계속 처리**를 허용한다.

3. DispatcherServlet 및 컨트롤러 호출  
    DispatcherServlet이 호출되어 HTTP 요청을 기본 컨트롤러(여기서는 PostController)로 라우팅한다.

4. 서비스 레이어 트랜잭션  
    PostController는 PostService를 호출하여 Post 엔터티 목록을 가져온다.  
    PostService는 새로운 트랜잭션을 시작하며, HibernateTransactionManager는 **OpenSessionInViewFilter에서 열린 동일한 Hibernate 세션을 재사용한다.**

5. 데이터 액세스 레이어  
    PostService는 PostDAO (데이터 액세스 객체)에게 Post 엔터티 목록을 가져오도록 위임한다.  
    PostDAO는 **어떠한 게으른 연관성도 초기화하지 않고 Post 엔터티 목록을 검색한다**. 게으른 연관성은 Hibernate에서 즉시 가져오지 않고 필요할 때 로드되는 관계이다.

6. 트랜잭션 커밋  
    PostService는 기본 트랜잭션을 커밋한다. 그러나 세션이 외부에서 열렸기 때문에( OpenSessionInViewFilter에서), **이 시점에서 세션은 닫히지 않는다**.
>[!failure] 주의
> 트렌젝션은 커멧 되더라도 오픈 세션(영속성 컨텍스트) 은 닫히지 않는다!


7. 뷰 렌더링 시작  
    DispatcherServlet이 사용자 인터페이스 (UI)를 렌더링하기 시작한다.

8. 게으른 연관성 초기화  
    렌더링 과정 중에 UI는 Post 엔터티의 게으른 연관성을 탐색한다.  
    이 탐색은 게으른 연관성의 초기화를 트리거하며 **추가적인 데이터베이스 쿼리**를 날린다.
**Lazy Loading**

9. 세션 닫힘  
    OpenSessionInViewFilter는 이제 Hibernate 세션을 닫을 수 있습니다. 렌더링 프로세스가 완료되었기 때문이다.  
    세션과 관련된 데이터베이스 연결이 해제된다.