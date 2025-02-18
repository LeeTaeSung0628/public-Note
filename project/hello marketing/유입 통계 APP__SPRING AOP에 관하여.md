---

---

# 👩‍👧‍👦유입 통계 APP__SPRING AOP에 관하여

#프로젝트 #개발 #DB

---


# 인입 페이지 주소 예시

https://www.hellofunding.co.kr/sp/loan/gtLoan?p=a2FrYW8xc3QK
https://www.hellofunding.co.kr/sp/loan/gtLoan?p=a2FrYW8xc222

# 구상
## 1. 광고 url 진입자, 파라미터(CODE) 쿠키 저장
- 통계 필요 페이지 내에서(프론트), 쿠키 데이터 페이지 별 최초 진입 확인?
## 2. Hit체크 하고싶은 페이지에서, CODE별 최초 진입 확인시 Hit로그 저장 controller 호출

## 3. 파라미터로 CODE 및 현재 페이지 주소(ex. /sp/loan) 저장

---


# 데이터 테이블 구상

| KEY `idx` | `hitDate`              | `hitCode`    | `hitUid`                             | `pageURL`               | `pageType` |
| --------- | ---------------------- | ------------ | ------------------------------------ | ----------------------- | ---------- |
| 3         | 2024-12-10 11-45-12.13 | a2FrYW8xc3QK | f2843ad6-2de0-48d2-9b5d-3b0eb4f9a4f5 | sp/loan                 | GET        |
| 2         | 2024-12-10 12-40-11.22 | AAA22dfxdfw2 | 94107590-2b39-478f-9bb3-32f4a8ef68cb | sp/loan/mortgage/notice | GET        |
| 1         | 2024-12-09 12-40-11.22 | aWfeYW8xc222 | 3eac413b-a95b-44f8-8f12-32ec4be98e39 | sp/marketing/hitTest1   | POST       |

# 고려사항
## 1. 추적을 얼마나 디테일하게 저장할지?
- 해당 진입시점부터 특정 동작에 대한 모든 타임라인 로그
## 2. 디테일에 대한 부하분산은 어떻게 할지?
- 레디스 가용 메모리에 대한 한계값 산정. -> 아직 적용 X
## 3. 인입시점이 아닌 동작에대한 unique값을 어떻게 지정할지?
- URL + 함수명 조합

# 정리

### 1. 첫 외부 URL진입 시점엔 controller redirect로직 으로 ==로그 쌓기== / ==쿠키 셋팅== => 이후 AOP 공통로직 화
=> 메인 테이블
=> 백로직에서 쿠키 데이터 쌓기
### 2. 이후 동작에 대해 ==쿠기값 비교== / ==AOP 로그 쌓기 공통 실행==
=> 디테일 테이블 - 타임라인으로 관리 단, 메인테이블에 존재하는 내셕들에 대해서
=> AOP에서 특정 서비스or메서드orURL로 지정하여 로그 쌀기

## **Spring AOP 사용 이유**
	- 관심사(Aspect)를 분리하여, 각 서비스 메서드에 반복해서 구현하는 것이 아닌, 별도의 Aspect로 관리하여 핵심로직을 공통으로 적용하기 위함이다.

#### 어떻게 자신의 메인 테이블을 찾을것인가? 
- 1. 난수 생성 후 물고있기☑
- 2. IP로 추적

---

# Path를 명시적으로 설정하여 주지 않았을 때, 쿠키가 등록되지 않는 이유
![[Pasted image 20241224144309.png]]

쿠키가 필요한 페이지의 경로가 기본 `path`와 일치하는 경우(`redirect url` 이 `SP_MARKETING_HIT_TEST1` 의 하위 url일 경우)
에는 명시적으로 표시할 필요가 없지만,
## 현재는 하위 URL이 아니기 때문에 경로를 명시적으로 설정해주어야 한다.


---

# AOP에서 Front-end 단의 특정 동작 필터링 하기


## 1. Pointcut 객체에 동적으로 enum에서 미리 선언한 값 execution으로 삽입하기
### **(이후 ADMIN 관리를 위해 enum객체 -> DB로 데이터 이전)**
![[Pasted image 20241226111559.png]]

---

# `@Pointcut` 어노테이션은 컴파일 시점에 고정된 문자열로 정의된 포인트컷 표현식을 기반으로 동작한다.
- **장점**:
    
    - 코드가 간결하고 읽기 쉽다.
    - Spring의 AOP 인프라를 사용하여 메서드 인터셉션을 쉽게 구현할 수 있다.
- **단점**:
    
    - 포인트컷 조건은 컴파일 시점에 고정된다.
    - 복잡한 조건이나 동적으로 변경되는 조건을 처리하기 어렵다.
-> *`@Pointcut` 등의 조건에 부합하는 Bean객체를 컴파일 시점에 찾아내어 프록시를 감싼다.*

### 이를 해결할 방법은?

# 프록시 기반 동적 포인트컷 생성방식
- **동적 생성**:
    
    - 런타임에 프록시를 생성하여 포인트컷과 어드바이스를 동적으로 적용.
    - `StaticMethodMatcherPointcut` 또는 `DynamicMethodMatcherPointcut`을 사용하여 런타임 조건 기반으로 메서드 매칭.
- **장점**:
    
    - 런타임 조건에 따라 동적으로 포인트컷 생성 가능.
    - 복잡한 조건과 동적 필터링을 처리하기 용이.
    - Spring AOP가 아닌 순수 Java 프록시 방식도 지원.
- **단점**:
    
    - 코드가 복잡해지고 추가 구현이 필요.
    - Spring AOP와 동일한 수준의 간결성을 제공하지 않음.

-> *해당 프록시 객체를 `적용하고 싶은 Bean객체`에 매번 생성(등록)해주어야 함.*

---

# customAdvisor를 사용한 동적 포인트컷 생성방식
``` java
import org.springframework.aop.Pointcut;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AopConfig {

    @Bean
    public Pointcut customPointcut() {
        return new CustomPointcut();
    }

    @Bean
    public CustomAdvice customAdvice() {
        return new CustomAdvice();
    }

    @Bean
    public DefaultPointcutAdvisor customAdvisor(Pointcut customPointcut, CustomAdvice customAdvice) {
        return new DefaultPointcutAdvisor(customPointcut, customAdvice);
    }

    @Bean
    public DefaultAdvisorAutoProxyCreator proxyCreator() {
        return new DefaultAdvisorAutoProxyCreator();
    }
}

```
#### 위 코드의 `DefaultAdvisorAutoProxyCreator`객체가 조건에 부합하는 빈에 대해 자동으로 프록시를 생성하여 `Advice`를 적용한다.
- **`@Pointcut` 방식과 동일한 동작**:
    - Spring 컨테이너가 관리하는 모든 빈에 대해 조건을 평가하고 프록시를 자동 생성합니다.
- **프록시를 명시적으로 선언할 필요 없음**:
    - ProxyFactory처럼 수동으로 프록시를 생성하지 않아도 됩니다.
- **유연성과 효율성 향상**:
    - Spring AOP 인프라를 활용하므로, 관리가 용이하고 런타임 성능이 개선됩니다.

---

# `DefaultAdvisorAutoProxyCreator`는 어떻게 `customAdvisor`의 조건에 부합하는 빈을 찾나?
- **BeanPostProcessor**:
    
    - `DefaultAdvisorAutoProxyCreator`는 Spring의 `BeanPostProcessor`를 구현한 클래스이다.
    - Spring 컨테이너는 애플리케이션 컨텍스트에 등록된 모든 `BeanPostProcessor`를 자동으로 호출하여 빈의 초기화 전후 작업을 수행한다.
    - 이를 통해, Spring AOP는 빈 생성 단계에서 프록시를 생성하고 조건에 맞는 빈에 대해 어드바이저를 적용.
- **Advisor 탐색**:
    
    - `DefaultAdvisorAutoProxyCreator`는 Spring 컨텍스트에 등록된 **모든 Advisor를 자동으로 탐색**한다.
    - 이는 `DefaultAdvisorAutoProxyCreator`가 `BeanFactory`를 통해 컨테이너의 모든 `Advisor` 타입 빈을 조회하기 때문.
    - 탐색된 `Advisor`를 사용하여 각 빈의 메서드와 포인트컷 조건을 매칭.
- **Advisor와 빈의 매칭**:
    
    - 빈이 생성될 때, `DefaultAdvisorAutoProxyCreator`는 해당 빈의 메서드가 어떤 `Advisor`의 포인트컷 조건에 부합하는지 평가.
    - 조건에 부합하면, 해당 빈을 **프록시 객체**로 감싸고, 어드바이스를 연결.

---

# 위 방법의 문제점? 
## 클래스에 인터페이스가 있으면 JDK 동적 프록시, 인터페이스가 없으면 CGLIB 프록시
- 이렇게 맵핑이 되어야 하나, 이를 동적으로 탐지하지 못한다.
``` java
/*  
 * EnableAspectJAutoProxy > 프록시 적용시 인터페이스/클래스 여부 자동 판단  
 * 클래스에 인터페이스가 있으면 JDK 동적 프록시.  
 * 인터페이스가 없으면 CGLIB 프록시.  
 * */@Configuration  
@EnableAspectJAutoProxy  
public class AopConfig {  
    @Bean  
    public Pointcut customPointcut() {  
        return new CustomPointcut();  
    }  
  
    @Bean  
    public CustomAdvice customAdvice() {  
        return new CustomAdvice();  
    }  
  
    @Bean  
    public DefaultPointcutAdvisor customAdvisor(Pointcut customPointcut, CustomAdvice customAdvice) {  
        return new DefaultPointcutAdvisor(customPointcut, customAdvice);  
    }  
	/*  
	* @EnableAspectJAutoProxy를 사용해 등록했기 때문에 불필요.  
	* */  
	//    @Bean  
	//    public DefaultAdvisorAutoProxyCreator proxyCreator() {  
	//        return new DefaultAdvisorAutoProxyCreator();  
	//    }  
}
```

### @EnableAspectJAutoProxy 어노테이션을 사용한다면, 빈 최초등록시 자동으로 판단하여 해당 프록시 객체를 적용할 수 있다.

---

# 프록시?
### 프록시(Proxy)란?

`프록시(Proxy)`는 **대리자**라는 뜻으로, **다른 객체에 대한 인터페이스 역할을 하는 객체**를 말합니다. 프로그래밍에서 프록시는 실제 객체에 접근하기 전에 특정 작업(로깅, 보안, 트랜잭션 관리 등)을 수행하거나, 객체에 대한 접근을 제어하는 데 사용됩니다.

#### 핵심 개념:

- **대리 객체**: 프록시는 실제 객체에 대한 **중간다리** 역할을 합니다.
- **동작 조정**: 프록시는 요청을 가로채서 추가적인 작업을 수행하거나, 요청을 변형한 뒤 실제 객체에 전달할 수 있습니다.
- **AOP와 연관**: 프록시를 사용하면 코드를 변경하지 않고도 객체의 동작을 확장하거나 변경할 수 있습니다.

---

# StaticMethodMatcherPointcut에서 사용가능한 호출 고유정보
## 타겟 클래스 / 메서드명 / 파라미터 / 어노테이션

Method Name: loanerLoginPage
javax.servlet.http.HttpServletRequest 
org.springframework.web.bind.annotation.GetMapping 

## 메서드 검증 포인트 컷 추가 후 빌드 시간
![[Pasted image 20241226152219.png]]

---

# ENUM객체에, 고유한 정보를 넣어주면 자동 집계가 시작된다
![[Pasted image 20241226171158.png]]
### customAdvice에 등록된 invoke함수는 Pointcut의 조건에 부합하는 메서드의 정보만을 가져온다.

---

# 케이스 정리

## 1. Pointcut에 있는 메서드이며, `"p"파라미터`가 있을때,
### 1.1 현재 쿠키에 `hitCode` / `hitUid` 가 없을때
- 외부링크 최초진입. hitCode, hitUid 발급 및 DB저장
### 1.2 현재 쿠키에 `hitCode` / `hitUid` 가 있을때
- 외부링크 재진입. hitCode는 변경될 수 있으므로 `"p"파라미터`로 재발급
=> `hitCode`는 매번 재발급 / `hitUid`는 없을때만 발급


## 2. Pointcut에 있는 메서드이며,`"p"파라미터`가 없을때,
### 2.1 현재 쿠키에 `hitCode` / `hitUid` 가 둘다 없을때
- 일반사용자 이며, 아무동작 하지 않는다.
### 2.2 현재 쿠키에 `hitUid` 또는 `hitUid` 가 있을때 
- 외부링크 진입 후 동작. 없는 녀석 새로 발급 및 DB저장

---

# 서비스 저장시 동시성 이슈가 발생할 수 있다?
``` java
@Override  
@Transactional  
public ResponseModel insertMarketingHitLog(String hitCode, String hitUid, String pageUrl, String pageType){  
    try {  
        Optional<HfMarketingHitLog> existingLog = hfMarketingHitLogRepository.findByHitCodeAndHitUidAndPageUrlAndPageType(hitCode, hitUid, pageUrl, pageType);  
        if (existingLog.isPresent()) {  
            HfMarketingHitLog logToUpdate = existingLog.get();  
            logToUpdate.setUpdateDate(LocalDateTime.now());  
            hfMarketingHitLogRepository.save(logToUpdate);  
            return new ResponseModel(ResponseModel.ResponseStatus.SUCCESS);  
        }  
        HfMarketingHitLog hfMarketingHitLog = HfMarketingHitLog.builder()  
                .hitCode(hitCode)  
                .hitUid(hitUid)  
                .pageUrl(pageUrl)  
                .pageType(pageType)  
                .build();  
        hfMarketingHitLogRepository.save(hfMarketingHitLog);  
        return new ResponseModel(ResponseModel.ResponseStatus.SUCCESS);  
    } catch (Exception e) {  
        return new ResponseModel(ResponseModel.ResponseStatus.FAILED, "데이터 처리 중 오류 발생");  
    }  
}
```

#### 1. **중복 데이터 조회와 저장 사이의 타이밍 차이**

- 여러 쓰레드(또는 트랜잭션)가 `findByHitCodeAndHitUidAndPageUrlAndPageType` 메서드를 호출하여 동일한 조건의 데이터를 동시에 조회할 수 있다.
- 두 쓰레드가 모두 `existingLog.isPresent()` 조건에서 `false`를 확인한 후, 동시에 새로운 `HfMarketingHitLog` 객체를 생성하고 저장하려 하면 데이터 중복 문제가 발생할 수 있다.

## 해결방법
. Lock을 쓰면 되나, 이후 업데이트 로직 삭제(로그 테이블화) 변경으로 처리

---

# 결과
![[Pasted image 20241230135801.png]]
* 1일의 유효기간을 갖는 uid를 발급하여, 인입코드 / Hit된 기능 주소 / 시간 을 저장한다.
* *uid는 외부url로 접근시 발급*
* `page_url`는 코드에 미리 등록 / `hit_code`는 모두(only 영문+숫자) 수용 가능


---

# 이후 PointCut에서 service 생성자 주입으로, DB에서 값로드 형식으로 변경
![[Pasted image 20250102160213.png]]
![[Pasted image 20250102160223.png]]

---
# ENUM객체에서 DB로 전환

1. enum객체에 정의된 메서드를 DB로 전환하여 pointcut에서 해당 클래스, 메서드를 가져와 조건으로 정의
2. 해당 조건에 부합하는 메서드를 프록시로 전환

### 다음 방법을 통해 DB로 admin과 app을 모두 관리 가능하도록 수정.

---
# AOP위빙 방식을 활용해, 동적으로 원하는 시간에 Pointcut에 조건에 해당하는 메서드를 다시 설정할 수 있을까?

 - api호출을 통해 advice내의 동작은 런타임 환경에서 동적으로 변경이 가능한 것을 확인했다.

## pointcut도 런타임에 변경이 가능하잖아? 런타임 위빙이니까?
- 런타입 위빙 방식이라고 하더라도, 변경은 불가능하다.
- 컴파일 위빙 방식은 컴파일 단계에서 원본 클래스 바이트코드를 변경하여 직접 위빙을 하는 방식이고,
 런타임 위빙은 ==런타임단계==에서 원본 클래스를 변경하지 않고 프록시 객체를 사용하여 교체하는 방식이다.
#### 이때, ==런타임단계== 라고 하지만 applicationContext가 bean들의 의존성주입을 마치기 이전에 동작하게 된다. 즉, 임의로 나중에 변경할 수 가 없다는 말이다.
##### * 그렇기 때문에, pointcut에서 @Autowired를 통해 service를 주입받지 못하였다. *


## 해결방법은 없을까?
### 1. appcontext에서 빈 전체 초기화
#### bean을 모두 refresh한다면 의존성이 깨지지 않고 런타입환경에서 AOP를 재적용 할수 있지 않을까?
=> 해당하는 부분은 리빌드와 큰 차이가 없고, 각 class에 scope를 따로 적용해야했기 때문에 고려하지 않았다.

### 2. 원하는 bean객체만 초기화
#### 의존 관계가 깨지는 영향도를 고려 할 수 없다.
=> 연관성 있는 모든 참조를 수동으로 재설정해야하기 때문에 매우 복잡해진다.

### 3. Filter를 사용한다.
#### 종단으로 나두는 AOP에서 횡단으로 나누는 필터를 적용한다면 적용 범위가 전체 범위로 확장되어 오버헤드가 크게 증가할 수 있다.

##### 해결방법이 떠오르지 않는다. 애초에 bean객체를 생성하는 시점에 맺어지는 의존관계를 전부 고려하며 다른 객체(프록시)로 변경하는 것은 쉽지 않다(권장하지도 않는다.)

## 결론
- 참조를 갱신하기 위해서는 Java리플렉션을 사용하여 강제로 참조를 갱신하거나, AspectJ의 런타임 위빙 방식을 활용해야 한다.
- 따라서, 서버 Kill을 하는 것과 진배 없다.
