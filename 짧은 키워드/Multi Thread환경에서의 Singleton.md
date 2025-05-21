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

