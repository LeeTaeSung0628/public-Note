# 🐡 Spring Security와 Filter

#프로젝트 #개발 #SPRING #Security #Filter

---

# <font color="#9bbb59">Spring Security 개요</font>

- Spring Security는 Java기반 웹 어플리케이션의 보안을 제공하는 강력한 프레임워크다.
- 웹보안(HTTP인증, 접근제어) 뿐 아니라, 메소드 레벨 보안, 세션 관리, JWT 등 다양한 보안 시나리오를 제공한다.
- Spring Security는 **Filter기반 아키텍처**를 사용한다.

Filter를 통해 HTTP요청과 응답을 가로채어 인증(Authentication), 인가(Authorization) 과정을 수행한다.

### Filter 기반 아키텍처

먼저, 스프링 시큐리티가 동작하기 위해서는 Filter가 필요하다.
여기서 Servlet Filter는 Spring의 관리에서 벗어나기 때문에, DI사용에 한계가 있다.
## ▶ [[Servlet]]이란??

![[do-messenger_screenshot_2025-04-29_15_37_33.png]]
이에 따라 스프링은, 요청을 위임하는 **DelegatingFilterProxy** 를 만들었다.

---

## DelegatingFilterProxy를 사용하는 이유

- *Servlet Container*(Tomcat 등..)는 오직 *Servlet Filter*만 인식할 수 있다.
이때, Spring Security의 인증, 인가 로직은 Spring Bean으로 만들어진다.

즉, 
- Tomcat입장에서는 Spring Bean을 직접 호출할 방법이 없다.
- Spring입장에서는 *Servlet Filter Chain*에 진입하여야 한다.

이 두개를 연결해 주는 역할을 하는것이 **<font color="#9bbb59">DelegatingFilterProxy</font>** 이다. 

#### 구체적인 요청 흐름
1. 클라이언트가 서버로 HTTP 요청을 보낸다.
2. 톰캣(Servlet Container)이 **Filter Chain**을 조회하여 **DelegatingFilterProxy**를 호출한다.
3. DelegatingFilterProxy는
    - Spring ApplicationContext에서 자신이 참조할 Bean 이름(보통 `"springSecurityFilterChain"`)을 찾는다.
    - 해당 이름의 Bean을 가져온다.
    - 가져온 Bean(보통 FilterChainProxy)의 `doFilter()` 메서드를 호출하여 실제 보안 처리를 진행한다.
4. FilterChainProxy는 내부에 등록된 다수의 SecurityFilter 들을 순서대로 호출한다.

이때 DelegatingFilterProxy가 직접 보안 로직을 수행하는 것이 아닌,
"Spring의 필터야, 이 요청 처리 좀 해줘" 하고 넘긴다.

```c
[ Client Request ]
       ↓
[ Servlet Container Filter Chain ]
       ↓
[ DelegatingFilterProxy (web.xml 등록) ]
       ↓ (Spring Bean 호출)
[ FilterChainProxy (Spring Security 설정에 따라 동작) ]
       ↓
[ Security Filter1 → Security Filter2 → ... → Security FilterN ]
       ↓
[ DispatcherServlet ]
       ↓
[ Controller 처리 ]
```

---

# <font color="#9bbb59">Spring Security의 주요 개념</font>
<br/>

## 1. 인증(Authentication)과 인가(Authorization)
<br/>

### **Authentication (인증)**  
    사용자가 누구인지를 확인하는 과정.  
    ex) 아이디와 비밀번호로 로그인하여 사용자를 식별.

1. 사용자가 로그인 폼을 통해 아이디와 비밀번호를 제출
2. `UsernamePasswordAuthenticationFilter`가 이 요청을 가로챔
3. 사용자 입력값을 바탕으로 `UsernamePasswordAuthenticationToken` 생성
4. `AuthenticationManager`에 인증 요청을 위임
5. `AuthenticationProvider`가 `UserDetailsService`를 통해 사용자 정보를 로드
6. `PasswordEncoder`로 비밀번호 비교
7. 인증 성공 시 `Authentication` 객체를 `SecurityContextHolder`에 저장


### **Authorization (인가)**  
    인증된 사용자가 어떤 리소스에 접근할 수 있는지 결정하는 과정.  
    ex) 일반 사용자는 `/admin/**`에 접근 불가, ROLE_ADMIN 만 접근 가능

1. 요청이 들어옴
2. `FilterSecurityInterceptor`가 요청 URL에 대한 접근 권한 체크
3. 인증된 `Authentication` 객체에 포함된 권한(`GrantedAuthority`) 목록과 비교
4. 인가 실패 시 `AccessDeniedException` 발생 → `AccessDeniedHandler`로 위임

<br/>

## 2. SecurityContext와 Authentication
<br/>

### SecurityContext

`SecurityContext`는 현재 사용자에 대한 **보안 정보를 저장하는 객체**다.  
Spring Security는 이 컨텍스트를 통해 현재 요청이 어떤 사용자에 의해 수행되는지 알 수 있다.

**저장 위치**:

- 기본적으로 **ThreadLocal**을 사용
- 이는 각 스레드마다 고유한 컨텍스트를 유지할 수 있도록 해준다

| 전략                            | 설명                    |
| ----------------------------- | --------------------- |
| `MODE_THREADLOCAL`            | 기본 설정. 스레드 단위 보안 컨텍스트 |
| `MODE_INHERITABLETHREADLOCAL` | 자식 스레드에게 보안 컨텍스트 상속   |
| `MODE_GLOBAL`                 | 전역 공유 컨텍스트            |
### Authentication 객체

Spring Security의 인증 상태를 나타내는 핵심 인터페이스다. 주요 구성 요소는 다음과 같다

| 메서드                 | 설명                                  |
| ------------------- | ----------------------------------- |
| `getPrincipal()`    | 사용자 정보 객체 (보통 `UserDetails`)        |
| `getCredentials()`  | 인증 수단 (보통 비밀번호, 성공 시 null 처리됨)      |
| `getAuthorities()`  | 권한 목록 (`ROLE_USER`, `ROLE_ADMIN` 등) |
| `isAuthenticated()` | 인증 여부                               |

### PasswordEncoder

Spring Security는 비밀번호를 **절대 평문으로 비교하지 않는다**.  
항상 `PasswordEncoder`를 통해 암호화된 상태로 비교해야 한다.

기본적으로 **BCryptPasswordEncoder**가 권장되며, 단방향 해시 함수로 안전하다.

```java
@Bean 
public PasswordEncoder passwordEncoder() {     
	return new BCryptPasswordEncoder(); 
}
```

인증 시 비밀번호 비교는 이렇게 이뤄진다
```java
String raw = "plain_pw"; 
String hashed = userDetails.getPassword(); 
boolean match = passwordEncoder.matches(raw, hashed);
```
