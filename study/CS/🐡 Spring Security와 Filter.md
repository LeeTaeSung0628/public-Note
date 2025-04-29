# 🐡 Spring Security와 Filter

#프로젝트 #개발 #SPRING #Security #Filter

---

# <font color="#9bbb59">Spring Security 개요</font>

- Spring Security는 Java기반 웹 어플리케이션의 보안을 제공하는 강력한 프레임워크다.
- 웹보안(HTTP인증, 접근제어) 뿐 아니라, 메소드 레벨 보안, 세션 관리, JWT 등 다양한 보안 시나리오를 제공한다.
- Spring Security는 **Filter기반 아키텍처**를 사용한다.
	-Filter를 통해 HTTP요청과 응답을 가로채어 인증(Authentication), 인가(Authorization) 과정을 수행한다.

---

### Filter 기반 아키텍처

먼저, 스프링 시큐리티가 동작하기 위해서는 Filter가 필요하다.
여기서 Servlet Filter는 Spring의 관리에서 벗어나기 때문에, DI사용에 한계가 있다.
## ▶ [[Servlet]]이란??

![[do-messenger_screenshot_2025-04-29_15_37_33.png]]
이에 따라 스프링은, 요청을 위임하는 **DelegatingFilterProxy** 를 만들었다.

---

