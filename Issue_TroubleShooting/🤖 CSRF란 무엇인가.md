# 🤖 CSRF란 무엇인가

#공부 #Tokken #Security #SPRING #보안 #개선 #이슈 #CSRF

---

CSRF를 임의의 수로 집어넣어도 인증이 된다??

```java
       // csrf 처리  
//     http.csrf().ignoringAntMatchers(baseProperties.getConfig().getIgnoringAntMatchersPatterns()); // 로그인 csrf 허용 추가
```
해당코드를 주석해도 인증이 된다??

# -> *우리 서비스느느 CSRF를 사용하고 있지 않나??*
# -> *아니면, 버프슈트 이후에 다시 덮어쓰나??*