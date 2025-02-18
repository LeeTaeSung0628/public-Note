
# 🐞etc

#공부 #기타

---

# Service의 순환참조를 막는 방법?
### 순서를 지켜야하는 service호출이있을때, 컨트롤러에서 서비스를 차례대로 호출하는것이 아니라 서비스를 주입받는 서비스를 만들어 호출하는 이유?
	순서를 맞추어 호출하는 로직과 예외처리를하는 로직이 컨트롤러에  집중되어 있다면 가독성이 떨어지게된다 
	이를 해결하기위해, 서비스에서 로직을 구현하게 된다면 서비스에서 서비스를 호출하는 순환참조를 야기할 수 있게된다.

	이를위한 해결법으로 서비스를 주입받는 메인 서비스를 만들어 여러서비스를 주입받고 한개의 서비스에서 이를 동작시킬 수 있다.


---


# Java에서는 두 가지 방법으로 문자열을 만들 수 있다.
``` java
	1. String x = "abc";
	2. String y = new String("abc");
```
1번의 경우로 생성했을 때는 abc라는 문자열을 String 상수 pool에 저장하고, 
다음번에 동일 문자열이 선언될 때 이풀에서 꺼내의 재사용하게 된다.

2번의 경우엔 String을 인스턴스와 하여 새로운 객체를 생성하게 된다.
String Class는 자신을 수정하는 기능을 제공하지 않기때문에,
1번의 경우로 선언했을경우 한가지가 바뀌게 되면 나머지가 모두 바뀌게 된다.
이러한 일을 방지하기 위해서는 생성자를 이용한 선언(2번)을 사용해야한다.


---


# Maven VS Gradle

### Maven
- Apach에서 2004년 출시한 빌드 툴이다.
- Ant를 사용하던 개발자들의 불편함을 해소 + 부가기능을 추가 하기위해 만들어졌다.

### Gradle
- Ant와 Maven의 장점을 모아 2012년 출시한 빌드 툴이다.
- Gradel이 시기 상 늦게 출시된 만큼 사용성/성능 등 비교적 뛰어난 스펙을 갖고 있다.

## Gradle이 Maven보다 좋은점
1. Gradle의 Groovy를 이용해서 기존 XML로 작성되있던 요소들의 단점을 해소하고 있다.
	- XML의 경우 코드가 길어지면 가독성이 떨어진다.
	- 의존관계가 복잡한 프로젝트 설정에 어려움이 있다.

2. 특정 상황에서 Gradle의 속도는 Maven보다 훨씬 빠르다.
	- Gradle은 캐시를 사용하기 때문에 반복될 수록 속도 차이는 더욱 커진다.


---


# Multi Thread환경에서의 Singleton
- 일반적으로 하나의 인스턴스만 존재해야 할 경우 Singleton패턴을 사용하게 된다.
single thread환경에서 사용되는 경우에는 아무런 문제가 없지만, Multi thread환경에서 
singleton객체에 접근 시 초기화 관련하여 문제가 있다.

- 보통 Singleton객체를 얻는 Static메서드는 getInstance( )로 작명하는게 일반적이다.

 ##### 그렇다면 singleton객체를 생성하는 로직을 어떻게 thread safe하게 적용할 수 있을까?
 - 단순하게 문제를 해결하고자 한다면, 메서드에 ==synchronized== 키워드만 추가해도 무방하다.
 하지만, 이는 하는 역할에 비해서 동기화 오버헤드가 심하다는 단점이 있다.

### LazyHolder
- 간단하게 설명하면, 객체가 필요할 때로 초기화를 미루는 것이다.

``` java
public class Singleton {
  private Singleton() {}
  public static Singleton getInstance() {
    return LazyHolder.INSTANCE;
  }
  
  private static class LazyHolder {
    private static final Singleton INSTANCE = new Singleton();  
  }
}
```
처음 singleton로딩 시에는 LazyHolder클래스의 변수가 없기 때문에 초기화 하지 않는다.
LazyHolder클래스는 singleton클래스의 getInstance( ) 메서드가 참조되는 순간 class가 로딩되며 초기화 된다.

Class를 로딩하고 초기화하는 시점은 thread-safe가 보장되기 때문에, 성능과 안정성을 모두 보장하는 훌륭한 방법이다.

