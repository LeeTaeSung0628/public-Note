# Servlet이란 무엇인가

**Servlet**은 간단히 말하면,

> "**자바로 작성된 웹 요청/응답을 처리하는 서버 측 프로그램**"이다.

- **Servlet** = **Server** + **Applets**의 합성어.
    
- 웹 브라우저의 요청(Request)을 받고, 그에 대한 응답(Response)을 생성하는 _자바 클래스_.
    
- **HTTP 통신 기반**으로 동작한다. (하지만 이론상은 TCP 기반도 가능)
    

**공식적 정의**:

> 서블릿은 Java 언어로 작성된 서버 측 컴포넌트로서, HTTP 요청을 받아 처리하고, HTTP 응답을 생성하는 역할을 한다.  
> (Java Servlet Specification, 현재 버전은 6.0)

---

- **서버 시작 시 또는 요청 시 로딩**
    
    - 톰캣(Tomcat) 같은 **서블릿 컨테이너**가 Servlet 클래스를 메모리에 로딩하고 객체를 생성한다.
        
- **`init()` 호출**
    
    - 생성된 서블릿 인스턴스에 대해 `init(ServletConfig config)` 메서드가 1회 호출된다.
        
    - 초기화 작업(리소스 연결 등)을 한다.
        
- **요청마다 `service()` 호출**
    
    - 클라이언트 요청이 들어오면 `service(ServletRequest req, ServletResponse res)`가 호출된다.
        
    - 여기서 HTTP 요청 종류(GET, POST 등)에 따라 알맞은 메서드(doGet, doPost 등)가 분기 호출된다.
        
- **서버 종료 시 `destroy()` 호출**
    
    - 서버 종료나 재배포 시, `destroy()` 메서드가 호출되어 리소스 정리(clean-up)한다.

---

## 서블릿 컨테이너가 뭐지?

**Servlet 컨테이너**는 Servlet의 생명주기를 관리하고 HTTP 요청을 대신 받아 Servlet에 연결해준다.

컨테이너가 하는 일은:

- HTTP 요청 수신
    
- URL 매핑 → 해당 Servlet 호출
    
- Servlet 인스턴스 생성/관리
    
- 멀티 스레드로 각 요청 처리
    
- Servlet API 제공 (`HttpServletRequest`, `HttpServletResponse` 등)
    

대표적인 Servlet 컨테이너: **Apache Tomcat**, Jetty, Undertow, WildFly 등