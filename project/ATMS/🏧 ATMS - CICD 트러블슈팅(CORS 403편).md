# 🏧 ATMS - CICD 트러블슈팅(CORS 403편)

#프로젝트 #트러블슈팅 #CORS #Spring #SpringSecurity #AI #ClaudeCode

---

<br>

지난 시간에는 Jenkins에서 배포 서버로 넘어가는 SSH 인증 문제를 네 단계 만에 해결했다.

## ▶ [[🏧 ATMS - CICD 트러블슈팅(SSH Permission Denied편)]]

배포는 성공적으로 끝났다. 컨테이너 7개 모두 `Up` 상태. 그런데 브라우저로 접속해서 로그인을 시도하자 곧바로 두 번째 미스터리가 나타났다.

---

<br>

# <font color="#76923c">증상 — 403인데, 백엔드 로그가 한 줄도 없다</font>

<br>

nginx 로그에는 403이 찍히는데,

```
"POST /api/auth/login HTTP/1.1" 403 ... "http://10.0.20.15:19080/login"
```

정작 `atms-web` 컨테이너 로그에는 **아무 것도 남지 않았다.** 요청이 아예 컨트롤러까지 도달하지 못했다는 뜻이다.

>[!failure] 가장 헷갈렸던 지점
> 인증(JWT) 문제인 줄 알고 토큰 발급 로직부터 의심했다. 하지만 로그인 API 자체가 호출된 흔적조차 없다는 게 이상했다 — **인증 이전 단계에서 막히고 있다.**

---

<br>

# <font color="#76923c">AI와 함께 요청 흐름을 한 줄씩 따라가 보기</font>

<br>

Claude Code에게 증상을 그대로 전달했다.

> "로그인 API 호출 시 403이 뜨는데, Spring 쪽 로그가 전혀 안 남아요. 컨트롤러 진입 전에 뭔가 막고 있는 것 같은데 어디부터 봐야 할까요?"

돌아온 방향은 명확했다 — **"Spring Security의 Filter Chain을 의심하라. 컨트롤러는 필터 체인을 통과해야만 실행되고, 그중에서도 CorsFilter는 체인 맨 앞쪽에서 동작한다."**

그 말을 따라 요청 흐름을 그려봤다.

```c
graph TD
	A[브라우저 fetch POST, Origin: http://10.0.20.15:19080]
	B[nginx 프록시 - Origin 헤더 그대로 전달]
	C[Spring Security CorsFilter]
	D{Origin이 허용 목록에 있는가}
	E[컨트롤러 진입]
	F[403 즉시 반환 - 로그 없음]

	A --> B --> C --> D
	D -->|Yes| E
	D -->|No| F
```

`SecurityConfig.java`를 열어보니 예상대로였다.

```java
// 허용 Origin이 코드에 하드코딩되어 있었다
configuration.setAllowedOrigins(List.of(
    "http://localhost:3000",
    "http://localhost:5173"
));
```

로컬 개발용 Origin만 등록되어 있었고, 실제 접속 주소(`http://10.0.20.15:19080`)는 목록에 없었다. **CorsFilter가 컨트롤러보다 먼저 요청을 끊어버렸기 때문에 애플리케이션 로그가 남을 기회조차 없었던 것**이다.

---

<br>

# <font color="#76923c">해결 — 하드코딩을 걷어내고 환경변수로</font>

<br>

로컬/서버 환경마다 Origin이 달라질 수밖에 없으니, 코드에서 완전히 빼서 환경변수로 주입하도록 바꿨다.

**① `SecurityConfig.java`**
```java
@Value("${cors.allowed-origins:http://localhost:3000,http://localhost:5173}")
private String corsAllowedOrigins;

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    List<String> origins = Arrays.stream(corsAllowedOrigins.split(","))
            .map(String::trim)
            .toList();
    configuration.setAllowedOrigins(origins);
    configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    configuration.setAllowedHeaders(List.of("*"));
    configuration.setAllowCredentials(true);
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    return source;
}
```

**② `docker-compose.yml`** — atms-web 환경변수 추가
```yaml
environment:
  CORS_ALLOWED_ORIGINS: ${CORS_ALLOWED_ORIGINS:-http://localhost:3000,http://localhost:5173}
```

**③ 서버 `.env`** — 실제 접속 주소 지정
```env
CORS_ALLOWED_ORIGINS=http://10.0.20.15:19080
```

atms-web 컨테이너만 재빌드·재시작하니 그제서야 로그인 응답이 200으로 돌아왔고, 컨테이너 로그에도 요청이 정상적으로 찍히기 시작했다.

---

<br>

**<font color="#76923c">결론</font>**
- JWT 인증 로직 문제 X
- nginx 프록시 설정 문제 X
- <u>**Spring Security CorsFilter의 하드코딩된 Origin 목록 O**</u>

"로그가 안 남는다 = 애플리케이션 코드에 문제가 없다"가 아니라, 오히려 **필터 체인처럼 컨트롤러 이전 단계에서 막히고 있다는 신호**일 수 있다는 걸 배웠다. AI에게 증상을 던지고 "어느 계층에서 막히고 있는지"부터 좁혀나간 게 삽질 시간을 크게 줄여줬다.

---

<br>

이렇게 파이프라인 구성 → SSH 인증 → CORS까지, ATMS의 CI/CD 구축과 초기 트러블슈팅을 마무리한다. Prometheus + Grafana로 모니터링을 붙이는 이야기는 다음 기회에 정리해본다.
