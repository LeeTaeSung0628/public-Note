# Spring WebFlux란?
- 스프링5에서 소개된 리엑티브 프로그래밍, 반응형 및 비동기적인 웹 어플리케이션 개발을 지원하는 모듈이다.
![[Pasted image 20240611144129.png]]

*리엑티브 프로그래밍이란 ?
	- 비동기  및 이벤트 기반 애플리케이션을 개발하기 위한 패러다임으로, 주로 높은 확장성과 성능을  제공하는것

내부통신을 이용하여 API프로젝트와 통신할때 주소 맵핑이 어떻게 이루어지는지??
	- Spring WebClient를 이용하여 내부 통신을 한다.
 
## Spring WebClient란??
- SpringWebFlux의 일부로써, 비동기적인 방식으로 HTTP 요청을 보내고 응답을 받을 수 있는 라이브러리이다.
- 웹으로 API를 호출하기 위해 사용되는 HTTP Client모듈 중 하나이다.
- RestTemplate과 같은 기능을 하지만, RestTemplate는 Blocking 방식이고, WebClient는 Non-Blocking방식이다.
Blocking 동기 - Non-Blocking 비동기 ( 정확히 같은것은 아니지만 비슷하다 ? )

- 요청자(APP)에서 WebClient라이브러리를 사용한 senderUtils를 사용하여
프로퍼티 소스와, 송신방식(GET/POST), 넘길 값(DTO), request를 수신할 값(ApiResponseModel)을 설정한다.

#### *Spring MVC환경에서도 SpringWebClient를 사용해도 될까?(현재 프로젝트가 이렇게되있다)
-SpringMVC에서는 WebFlux와 달리, 블로킹I/O를 사용하기 때문에, 동기적인 작업을 수행할 떄에는 WebClient보다 RestTemplat이 효과적이지만, 비동기 작업을 할 때에는 WebClient를 사용하는것이 효과적이다.

---

# Spring WebClient VS RestTemplate

### RestTemplate : Multi-Thread / Blocking방식

- Thread Pool을 애플리케이션 구동시 미리 만들어 두고,
 요청시 가용한 Thread가 있다면 1요청당 1Thread가 할당된다.
- 만약 가용한 스레드가 부족하다면 Queue는 대기하게 되며,
 전체 서비스의 속도가 현저히 느려지게 된다.

### Spring WebClient : Single-Thread / Non-Blocking 방식

- Core당 1개의 Thread를 사용한다.
- 요청은 Event Loop내의 job으로 등록되고, 각 job을 제공자에게 요청한 후,
 기다리지 않고 다른 job을 처리한다.
- Event Loop는 제공자로부터 callback으로 응답이 오면, 그 결과를 요청자에게 제공하낟.
- 따라서 반응성/탄력성/가용성/비동기성 을 보장하기 때문에 동시사용자가 크게 몰렸을때
RestTemplate에 비해 성능이 저하되지 않는다.


---