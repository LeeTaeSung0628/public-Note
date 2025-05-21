# 예외처리(Exception)

### 0. 에러 메세지로 부터 정보를 받는 메서드
	1. e.getMessage() : 에러 메시지의 정보를 받음
	2. e.getExceptionCode() : 에러 메시지 발생 코드를 받음
	3. e.getStatus() : 에러 메시지의 발생 상태를 받음
	4. HttpStatus(enum 클래스)를 받아 해당 value(Code) 와 getReasonPhrase(message)를 얻을 수 있음

### 1.  체크 예외(Checked Exception)와 언체크 예외(UnChecked Exception)
	1. 체크 예외
		발생한 예외를 잡아서(catch) 체크 후 예외를 복구 or 회피 하도록 만드는 구체적인 처리를 필요로 하는 예외이다.
		try catch가 강제된다. 컴파일 시점에서 에러의 확인이 가능하다.
		try catch를 할 수 없다면 예외를 밖으로 던지는 Throw 예외를 필수로 선언해 주어야 한다.
	2. 언체크 예외
		예외를 잡아서 해당 예외에 대한 처리가 필요 없는 예외.
		RuntimeException을 상속 받은 예외들이 이에 포함된다.

### 2. 예외처리 방법 3가지
#### 1. try~catch : 다른 작업 흐름으로 유도한다. checkedException으로 처리
#### 2. throws~ : 호출한 쪽(부모)에게 예외 처리를 위힘한다.
#### 3. throw~ : 명확한 의미의 예외로 바로처리, 개발자들이 비즈니스 로직에서 처리하는 방식
	UncheckedException으로 처리

## 1. try-catch
``` java
try {
	예외가 생길 가능성이 있는 코드
} catch (예외종류){
	예외처리 코드
} finaly {
	예외와 상관없이 항상 실행시킬 코드(선택사항)
}
```
자바에서는 Exception클래스에서 상속받은 다양한 Exception클래스를 갖고 있기 때문에, 여러가지 에러 발생 가능성에 대해서 예외 구문을 처리해 줄 수 있다.

## 2. throws

자신을 호출하는 메서드에 예외처리의 책임을 떠넘기는 것이다.
단, throws를 사용하려면 반드시 호출한 메서드에 try-catch 구문을 사용하여 예외를 처리해 주어야 한다.

``` java
public class ThrowTest {

    public static void main(String[] args) {

        int n1, n2;

        n1=12;
        n2=0;

        try {
            throwTest(n1, n2);
        } catch (ArithmeticException e) {
            // n1/n2 라면 발생했을 것
            System.out.println("ArithmeticException: " + e.getMessage());
        }
    }

    public static void throwTest(int a, int b) throws ArithmeticException{
        System.out.println("throw a/b: "+ a/b);
    }
}
```
## 3. throw
throw와 throws는 큰 차이가 있다.
throw는 개발자가 직접 예외를 발생시키고싶을 떄 사용하는 것이다.
주로 RuntimeException처리를 위해 사용한다.

**checkedException에서도 사용이 가능하다.
``` java
throw new IOException("IO Exception occurred");
```

사용예제
``` java
public class ThrowTest {

    public static void main(String[] args) {

        int n1, n2;

        n1=12;
        n2=0;

        try {
            throwTest(n1, n2);
        } catch (ArithmeticException e) {
            // n1/n2 라면 발생했을 것
            System.out.println("ArithmeticException: " + e.getMessage());
        }
    }

    public static void throwTest(int a, int b) throws ArithmeticException{
        throw new ArithmeticException();
    }
}
```
해당 코드의 익셉션 메세지는 null 로 뜨게 된다.
throw는 Exception을 던질 때, 예외 내용을 함께 던져 주지 않기  때문이다.
그래서 개발자가 Exception을 따로 커스터마이징해서 만들고, 그 안에 메세지를 넣어서 던져주는 방식이다.


### 결론
11. CheckedException => try ~ catch 문, throws(의존관계) 로 처리!

12. UnCheckedException(RuntimeException) => 기본적으로 복구 불가능한 예외(발생시 런타임 중지)로, CheckedExceptoin이어도 더 구체적인 UnCheckedException으로 발생시켜! throw로 exception을 던지고, ExceptionHandler로 처리!
13. 언체크드익셉션(런타입익셉션) -> 기본적으로 복구 불가능한 예외(발생시 런타임 중지)로, 체크드익셉션이어도 더 구체적인 언체크드익셉션으로 발생시켜 쓰로우로 익셉션을 던지고, 익셉션핸들러로 처리

---