# Java의 람다란?
- 람다 표현식(Lambda Expression)이란?
	함수형 프로그래밍을 구성하기 위한 함수식이며, 간단히 말해 ==자바의 메소드를 간결한 함수 식으로 표현==한 것.

- 이름없는 함수, ==익명 함수==(anonymous function) 이라고도 한다.

``` java
int add(int x, int y) {
    return x + y;
}
```

``` java
// 위의 메서드를 람다 표현식을 이용해 아래와 같이 단축 시킬수 있다. (메서드 반환 타입, 메서드 이름 생략)
(int x, int y) -> {
	return x + y;
};
```

``` java
// 매개변수 타입도 생략 할 수 있다.
(x, y) -> {
	return x + y;
};
```

``` java
// 함수에 리턴문 한줄만 있을 경우 더욱 더 단축 시킬 수 있다. (중괄호, return 생략)
(x, y) -> x + y;
```


---
# 자유변수
- 람다의 바디에서는 파라미터가 아닌 바디 외부에 있는 변수를 참조할 수 있다.
- 유사하게 로컬 클래스, 익명 클래스에서도 참조가 가능하다.
``` java
public class VariableCapture {
	private void run() {
    	// 로컬 클래스, 익명 클래스, 람다에서 이 변수를 참조하면 effective final로 변경
        int baseNumber = 10;
        
        // 람다
        IntConsumer lambda = (i) -> System.out.println(i + baseNumber); // i + 10
        
        // 로컬 클래스
        class LocalClass {
            void printBaseNumber() {
                System.out.println(baseNumber); // 10
            }
        }
        
        // 익명 클래스
        IntConsumer intConsumer = new IntConsumer() {
            @Override
            public void accept(int i) {
                System.out.println(i + baseNumber); // i + 10 
            }
        };
    }
}
```
##### 여기서, 람다 시그니처의 파라미터로 넘겨진 변수가 아닌, 외부에서 정의된 변수를
##### ==자유 변수==라고 한다. 또, 람다 바디에서 자유 변수를 참조하는 것을 
##### ==람다 캡쳐링==이라고 한다.

## 제약조건
12. 자유 변수는 `final`로 선언되어 있어야 한다.
13. `final`로 선언되지 않은 자유 변수는 `final`처럼 동작해야 한다. (effectively final)

### 왜 `final`이어야 할까?
- 지역 변수는 JVM의 영역 중 stack영역에 생성된다.
- 쓰레드별로 해당 stack영역을 별로 갖는다.
- 즉, 쓰레드 끼리 공유가 되지 않는다.
- 반면 인스턴스 변수는 Heap영역에 생성된다.
- 즉, 인스턴스 변수는 공유가 가능하다.
*람다는 별도의 쓰레드에서 실행이 가능하다.*
*따라서 지역 변수(자유 변수)가 있는 쓰레드가 사라졌을 때, 람다가 이 변수를 참조하고 있다면, 오류를 야기할 수 있는것이다.*

### 어떻게 해결했을까?
- 람다는 자유 변수를 참조할 때 직접 그 변수를 참조하는 것이 아닌,
 자유 변수를 자신의 ==stack영역에 복사==하여 참조하는 방법으로 참조 오류를 해결했다.
- 이러한 이유로 자유 변수는 수정이 불가능하도록 `final`처럼 동작해야 하는 것이다.

## 주의점
- 로컬 클래스 / 익명 클래스 / 람다 모두 자유 변수를 참조할 수 있다는 공통점이 있다.
- 하지만 로컬클래스 / 익명클래스와 다르게 람다에서는 자유 변수와 같은 이름의 변수를 선언할 수 없다.
*람다의 scope는 자유변수의 scope와 같기 때문이다.*
*반면 로컬/익명 클래스는 내부에 생성된 변수의 스코프가 더 지엽적이기 때문에 선언이 가능하다.*

---
