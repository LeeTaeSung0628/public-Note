
# ☕JAVA , OOP

#공부 #Java #OOP

---

# 오버로딩 / 오버라이딩

## 오버로딩 
	- 컴파일 다형성
	- 이름이 같지만, 매개변수의 타입/개수/순서 를 다르게 정의하여 사용하는 방법
	- 어떤 메서드가 호출될지 컴파일 시점에서 결정
	- 코드를 직관적이게 만드는데 사용
## 오버라이딩
	- 런타임 다형성
	- 부모클래스 또는 인터페이스에 정의된 메서드를 하위 클래스에서 재정의하여 사용하는 방법
	- 어떤 메서드가 호출될지 런타임 시점에 결정된다.
``` java
Map<String, Integer> map1 = new HashMap<>();
Map<String, Integer> map2 = new TreeMap<>();
```
해당 형태로 map1 / 2를 구현했다면.
객체 타입은 각각 HashMap/TreeMap 이 되며,
변수 타입은 모두 Map이 된다.
	- 즉 map1.add / map2.add 등 Map인터페이스가 가진 함수만을 사용할 수 있으며,
	HashMap이나 TreeMap가 가진 고유메서드는 사용할 수 없다.
	하지만, 오버라이딩(런타임 다형성)으로 재정의한 각각의 메서드로 해당 동작이
	구현체의 특성에 맞게 실행되게 된다.

---
# 상속과 합성
#### 상속의 개념
##### - extends : 부모에서 선언 및 전의를 모두 마치며, 자식은 메서드 / 변수를 그대로 사용 가능한 형태
	Java는 다중 상속을 지원하지 않는다.(부모가 2명 이상인것)
##### - impliments : interface구현. 부모 객체는 선언만 하며, 정의는 자식에서 오버라이딩 하여 사용하는 형태
	다중 상속 처럼 여러개를 상속받을 수 있다.  
	부모의 메서드를 사용하며, 동작이 의도대로 흘러가도록 강제할 수 있으나, 구현은 자식 클래스에서 하기때문에 결합도를 낮출 수 있다.
##### - abstract : extends와 impliments의 혼합. extends는 하되, 부모의 몇 개는 추상 메서드로 구현되어있는 형태

## 상속의 단점
1. 상속은 부모 클래스의 내부 구현에 대해 상세히 알아야 하기 때문에, 자식과 부모 사이의 결합도가 높아질 수 밖에 없다.
2. 또한, 부모의 쓸모없는 기능까지 모두 받게 될 가능성이 있다.
3. 부모 클래스가 수정되면, 자식클래스도 동시에 수정해야하는 경우가 생긴다.
4. 단일 상속만 가능하기 때문에, 결국 인터페이스를 또 사용하게 된다.


## '합성' 이란?
- 합성은 구현에 의존하지 않는 점에서 상속과 다르다.
	- 합성을 이용했을 때는, 객체의 내부는 공개되지 않고 인터페이스를 통해 코드를 재사용하기 때문에,
	구현에 대한 의존성을 인터페이스에 대한 의존성으로 변경하여 ==결합도를 낮출== 수 있다.
	- 합성 관계는 실행 시점에 동적으로 변경될수 있다.(런타임)
``` java
public class Phone {
    private RatePolicy ratePolicy; // 클래스 합성
    private List<Call> calls = new ArrayList<>(); // 클래스 합성

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }
}
```

#### 결론은 다형성을 이용하고 싶을때, 부모클래스를 정확히 이해하고 그대로 계승할것이 아니라면, extends 대신 impliments를 사용하여 결합도를 낮추는 것이 좋고, 그 외 모든 경우네는 ==합성==을 이용하도록 권고하고있다. 


---

# 객체지향설계 5원칙 S.O.L.I.D

## 이녀석들은 뭐하는 애들인가
- 객체지향 설계시 지켜야 하는 5가지 원칙들의 앞글자를 딴것이다.
- 순서는 노상관이다.

### 1. 단일 책임의 원칙 - 클래스 하나에 너무 많은 책임을 넣지마라. 근데 기준은 너가 정한다.
	책임 = 기능 이다. 수정시 영향도를 낮추는 주요한 원칙이다.
### 2. 개방 폐쇄의 원칙 - 확장에는 열려있게, 변경에는 닫혀있게 설계해라.
	- 추상화 사용을 통한 관계를 구축하기를 권장하는 의미이다.
	- 추상클래스,인터페이스를 통한 관계를 구축하여 확장은 쉽고, 변경에는 영향도가 없어지도록 분리해라?
### 3. 리스코프 치환의 원칙 - 하위타입은 언제나 상위타입으로 전환이 가능해야한다.
	- 코드에는 문제가 없더라도, 부모타입의 설계 목적에도 부합하도록 설계해라. -> 부모의 동작의도대로 흘러가도록 설계해라
	- Map a = new HashMap(); 같이 구현해서, 사용해도 문제 없도록 하는거다
	- > 변수는 Map타입으로, Map메서드만 사용가능하지만 객체는 HashMap이기 때문에 각각의 기능으로 수행되고, 수정시에도 Map을 상속받는 다른 클래스로 변경이 쉬워진다.
### 4. 인터페이스 분리의 원칙 - 인터페이스를 기능별로 잘 분리하고, 수정하지 말아라
	- 인터페이스의 단일책임과 비슷하다. 기능별로 잘 분리하고, 수정을 최소화할수 있도록 처음부터 생각하라.
### 5. 의존성 역전의 원칙 - 어떤 클래스를 참조할때, 직접참조하지 말고 상위 인터페이스나 추상클래스를 참조해라.
	- 리스코프 치환의 원칙을 따라 설계했다면, 의존성 역전의 원칙을 따르기 쉬워진다.
	- 의존관계를 맺을때, 변화기 쉬운것 보다, 변화하기 어려운 것에 의존하라는 것이다.


---

# JDK와 JRE
## JDK - Java Development Kit
- 자바 개발 키트의 약자로, 개발자들이 자바로 개발하는데 사용되는 SDK키트라고 생각하면 된다.
- 자바 개발시 필요한 라이브러리와, javac, javadoc 등의 개발도구를 포함한다.
- 자바 실행 프로그램인 JRE도 포함한다. (JRE에는 JVM이 들어가있다.)
##### ** SDK란? : Software Development Kit로 하트웨어플랫폼, 운영체제 또는 프로그래밍 언어 제작사가 제공하는 툴.
	- 대표적으로, 안드로이드 스튜디오 등이 있다.

## JDK 버전
- Java SE(standard edition) : 가장 기본이 되는 표준 에디션의 자바 플랫폼.
- Java EE(enterprise edition) : 대규모 기업용 에디션. SE의 확장판
- Java ME(micro edition) : 피쳐폰/셋톱박스/프린터 와 같은 작은 임베디드 기기를 다루는데 이용하는 에디션
- Java FX : 가볍고 예쁜 그래픽 사용자 인터페이스를 제공하는 에디션

#### 사람들이 JAVA를 설치한다고 하는것은 JDK를 설치한다고 할 수 있다.
##### JDK자체는 오픈소스이지만, 또다른 JDK버전을 여러회사에서 출시했고, 지금의 JDK환경이 갖춰졋다.


## JRE - Java Runtime Environment
- JRE는 자바 실행환경의 약자로서, JVM과 자바 프로그램을 실행시킬 때 필요한 라이브러리 API를 함께 묶어서 배포되는 패키지.
- 또한, 자바 런타임 환경에서 사용하는 프로퍼티 세팅과 리소스(jar)파일을 가지고 있다.
- JRE는 기본적으로 JDK에 포함되어있기 때문에 JDK를 설치하면 함께 설치된다.
![[Pasted image 20250110101642.png]]
### java로 프로그램을 직접 개발하려면 JDK가 필요하고, 
### 컴파일된 java프로그램을 실행시키려면 JRE가 필요하다.


## JVM - Java Virtual Machine
- JVM은 자바 가상머신의 약자로서, 직역하면 자바를 실행하는 머신, 자바를 돌리는 프로그램이다.
- 자바로 작성된 모든 프로그램은 JVM에서만 실행될 수 있으므로, 자바 프로그램을 실행하기 위해서는 바늗시 자바 가상머신의 설치가 선행되어야 한다.
#### JVM을 사용함으로써 얻는 가장 큰 이득은 ==자바프로그램을 모든 플랫폼에서 제약없이 동작==시킬 수 있는 부분이다.


## JVM이 왜 필요할까?
- java는 OS에 종속적이지 않다는 특징을 가지고 있다.
### c언어를 예로 든다면,
- 소스코드를 컴파일하여 기계어를 만드는 과정에서, window/mac/linux가 각기 다르게 컴파일을 시킨다.
- 때문에 각 OS별로 상이한 문법을 사용하게 되는 일이 벌어진다.
- 이러한 언어를 "이식성이 낮다" 라고 표한다.

- 하지만 java는 jvm를 거쳐서 OS와 상호작용 하기때문에, OS에 구애받지 않게 된다.

## JVM은 어떤 방식으로 동작하는가?
- 위에서 c언어는 컴파일을 거치면 기계어가 된다고 했는데,
- java는 JVM을 거쳐 ==바이트 코드==로 변환되게 된다. 
- 이는 가상 머신이 이해할 수 있는 중간 레벨의 언어로, 반쪽짜리 컴파일 결과물 이라고 할 수 있다.
- 이는 어떠한 환경에 종속적이지 않고 실행될 수 있다.
- 즉, 재컴파일 할 필요없이 기계가 바로 읽고 실행 할 수 있는 코드를 만들어 주는것이다.

*하지만 자바 프로그램과 달리 JVM은 각 운영체제에 종속적이므로, 각 운영체제에 맞는 JVM을 알맞게 설치해주어야한다.*


## JVM의 단점
- 위의 설명과 같이 java는 일반 프로그램보다 JVM이라는 단계를 한 단계 더 거치기 때문에, 상대적으로 실행속도가 느리다는 단점을 내포하고 있다.
- 이를 보환하기 위해, 필요한 부분만을 기계어로 바꾸어 속도를 향상시키는 JIT 컴파일러 같은 내부 프로그램이 있지만, 그럼에도 여전히 느리다.


## Java소스 컴파일 과정
![[Pasted image 20250110103409.png]]
- 위 그림에서 Compiler는 javac.exe에 해당되고 JVM은 java.exe에 해당된다.

1. 소스코드(MyPrograme.java)를 작성한다.
2. 컴파일러(Compiler)는 자바 소스코드를 이용하여 ==클래스 파일(MyProgram.class)==을 생성한다. 컴파일 된 클래스 파일은 JVM(Java Virtual Machine)이 인식할 수 있는 바이트 코드 파일이다.
3. JVM은 클래스 파일의 바이트 코드를 해석하여 바이너리 코드로 변환하고 프로그램을 수행한다.
4. MyProgram 수행 결과가 컴퓨터에 반영된다

##### 왜 컴파일을 마친 프로그램이 exe가 아니라, class파일일까?
- c또는 c++등으로 작성된 프로그램은 최종 결과물로 exe파일을 만들어낸다.
- java도 exe파일을 만들 수 있지만, class파일로 굳이 만들어내는 이유는 다음과 같다
	- JVM이 exe에 포함되는 형식으로 가능하기 때문에 exe파일이 무척 커지게 되는 단점이 있다.
	- 때문에 보통의 경우 일부러 생성하지 않는것이다.

---

# Java의 String
- java에서 String은 객체이다.
- int, char와 달리 기본형,원시형(primitive type)이 아닌 참조형(reference type)변수로 분류된다.
- 메모리의 Stack영역이 아닌, Heap영역에서 문자열 데이터가 생성되고 다뤄진다는 말이다.
![[Pasted image 20250110122312.png]]
- 또한 String은 불변(Immutable)객체이다.
	- 예를들어, s = "a"; 에 s = s + s; 를 하면 "aa"가 되겠지만, heap영역 메모리에 새로운 주소로 생성하게 된다.

## 왜 불변객체로 설계 되었을까?

### 1. 성능적 이득
- JVM에서는 ==String Constant Pool==이라는 독립적인 영억을 Heap영역에 구축하여
 문자열들을 Constant화 하여 다른 변수 혹은 객체들과 공유한다.
- 이 과정에서 ==데이터 캐싱==이 일어나고, 그만큼 성능적인 이득을 취할 수 있게된다.

### 2. 안정성
- 데이터가 불변하다면, 멀티 스레드 환경에서 ==동기화 문제가 발생하지 않기== 때문에 안전한 결과를 낼 수 있다.

### 3. 보안
- 만일 번지수의 문자열 값이 변경이 가능하다면, 참조값을 변경하여 애플리케이션에 보안 문제를 일으킬 수 있다.

## Java에서 String 주소 할당방식
#### 1. 문자열 리터럴을 이용한 방식 - String str1 = "안녕";
#### 2. new연산 이용방식 - String str2 = new String("안녕");
### 둘다 "안녕" 이라는 문자열을 저장하는 점은 같지만, JVM 내부 메모리 측면에서는 큰 차이가 있다.

- 먼저 문자열 리터럴 방식으로 변수에 저장하게 되면, 이 값은 string constant pool에 저장이 되지만, `new`연산자를 사용하여 생성한 값은 Heap영역에 존재하게된다.

## 비교 연산
### equals 
- 대상 값 자체를 비교

### ==
- 대상의 주소값을 비교.

즉, `new`연산자를 통해 만들어진 객체를 equals로 비교한다면 `true`가 나오겠지만, 
`==` 연산자를 사용한다면 `false`가 나오게 되는것이다.

---

# SpringBuffer와 SpringBuilder
 두 클래스 모두 문자열을 연산(추가 및 변경)할 때 주로 사용하는 자료형이다.
	 물론 String자료형 으로도 `+` 나 `concat()`으로 문자열을 이어붙일수 있다.
	 하지만 `+`를 이용해 String인스턴스의 문자열을 결합하면, 내용이 합쳐진 새로운 String인스턴스를 생성하게된다.
	 문자열을 많이 결합하면 결합할수록 공간낭비는 물론, 속도 또한 매우 느려진다.

이를 해결하기 위해 Java는 문자열 연산을 전용으로 하는 자료형을 따로 만들어 제공하였다.

## SpringBuffer란?
- 내부적으로 Buffur라고 하는 독립적인 공간을 가지게 되어
- 문자열을 바로 추가할 수 있어 공간의 낭비도 없으며, 문자열 연산 속도도 매우 빠르다

기본적으로 16개의 문자를 저장할 수 있는 크기이며, 
연산 중 할당된 버퍼의 크기를 넘게되면 자동으로 버퍼를 증강 시킨다.

#### 또한 다양한 내장 메서드를 지원하여, 문자열을 가공할 수 있다.
**SpringBuffer와 SpringBuilder의 메서드 사용법은 동일하다.**

## String  vs  (SpringBuffer와 SpringBuilder)

### String은 불변
- 불변자료형 으로써, 초기 공간과 다른 값에 대해서 새로운 메모리 공간을 할당하여 새로 생성한다.
- 그렇게 남겨진 문자열 값은 java가비지 컬렉터에 의해 제거될 대상에 포함된다.

### SpringBuffer와 SpringBuilder는 가변
- 즉, 문자열을 조작할 때 새 객체를 생서하지 않고 기존 객체를 수정한다.
- 메모리 관리 측면에서 효율적

| 특성           | **String**     | **StringBuffer**   | **StringBuilder**  |
| ------------ | -------------- | ------------------ | ------------------ |
| **가변성**      | 불변 (Immutable) | 가변 (Mutable)       | 가변 (Mutable)       |
| **스레드 안전성**  | 스레드 안전         | 스레드 안전             | 스레드 안전하지 않음        |
| **속도**       | 상대적으로 느림       | 상대적으로 느림 (동기화로 인한) | 빠름 (동기화 없음)        |
| **주요 사용 사례** | 읽기 전용 데이터 처리   | 멀티스레드 환경에서 문자열 조작  | 단일 스레드 환경에서 문자열 조작 |

#### 그렇다면 문자열 + 연산시에는 무조건 String을 사용하는것이 옳지 않은건가??
=> 사실 자바는 문자열에 + 연산을 사용하면, 컴파일 전 내부적으로 StringBuilder 클래스를 자동으로 생성한 후 다시 문자열로 돌려준다. 다만, 문자열을 합치는 일이 빈번할 경우에는 단순히 +연산을 사용하는것은 효율이 떨어지므로
SpringBuffer와 SpringBuilde 를 사용하는것이 옳다고 할 수 있다.

---

# AOP위빙 방식

- **컴파일 타임 위빙 (Compile-Time Weaving)**
    
    - 소스 코드 컴파일 시, Aspect가 대상 객체에 결합됩니다.
    - AspectJ 같은 프레임워크에서는 가능하지만, Spring AOP는 이 방식을 지원하지 않습니다.
- **로드 타임 위빙 (Load-Time Weaving)**
    
    - 클래스 파일을 JVM에 로드할 때 Aspect를 결합합니다.
    - Spring AOP는 기본적으로 지원하지 않으나, AspectJ 통합 설정을 통해 사용할 수 있습니다.
- **런타임 위빙 (Runtime Weaving)**
    
    - **Spring AOP의 기본 방식**입니다.
    - 런타임에 프록시 객체를 생성하여 부가 기능을 결합합니다.
    - JDK 동적 프록시 또는 CGLIB를 사용하여 대상 객체를 프록시로 감싸고, 프록시가 메서드 호출을 가로채서 Advice를 실행합니다.

### pointcut : advice를 적용시킬 조건 정의
### advice : pointcut으로 적용시킨 프록시에서의 동작 정의
### aspect : pointcut + advice
### weaving : AOP 가 aspect를 적용시키는 행위

---
# AOP의 PointCut 실행 시점?

- **프록시 생성 시**: 클래스 단위에서 PointCut 조건에 따라 프록시가 생성됩니다.
- **메서드 호출 시**: 개별 메서드 단위에서 PointCut 조건을 재평가합니다.

#### **PointCut 조건 재평가의 이유:**

1. 프록시 생성은 클래스 단위로 이루어지지만, PointCut 조건은 메서드 단위로 적용됩니다.
2. 런타임 정보(매개변수, 리턴 타입 등)를 기반으로 동적 조건을 평가해야 하는 경우가 있습니다.
3. Spring AOP의 유연성과 확장성을 보장하기 위한 설계입니다.

#### 결론 : 실행 시 1번 , 메서드 최초 실행 시 1번. 총 2번만 실행된다.
##### 단, Advice는 메서드 최초 실행시 조건에 부합하다면 매번 실행된다.

---

# 데코레이션 패턴 vs 프록시 패턴

## 프록시란?
- 클라이언트에서 서버를 직접 호출하고, 처리결과를 받는다. -> ==직접호출==
- 클라이언트에서 서버를 직접 호출하지 않고, 대리자를 통해 간접적으로 서버에 요청 -> ==간접호출==
#### 여기서, 간접호출하는 대상을 ==프록시== 라고 한다.

## 프록시는 클라이언트와 서버의 중간에 위치하기 때문에, 여러가지 일을 수행할 수 있다.
- 1. 권한에 따른 접근 차단, 지연로딩을 수행하는 ==접근 제어==
- 2. 서버의 기능에 다른 기능을 추가해주는 ==부가 기능 추가== ex) 로그, 가공
- 3. 대리자가 또 다른 대리자를 호출하는 ==프록시 체인==

## 프록시는 대체 가능해야 한다.
- 1. 아무 객체나 프록시가 되는것은 아니다.
- 2. 클라이언트는 서버에 요청한지, 프록시에게 요청한지 몰라야한다.
- 즉, 클라이언트의 코드를 건드리지 않고 프록시 추가와 ==런타임 객체 의존 관계 주입==만 변경하여야 한다.

## 프록시 패턴과 데코레이터 패턴
- 두 패턴 모두 프록시를 사용하는 방법이다.
- 또한, 둘 모두 원본 객체를 건드리지 않고, 추가 기능을 실행 할 수 있다.
- 하지만 GOF 디자인 패턴에서는 이 둘을 의도(Intent)에 따라서 구분한다.
	==프록시 패턴== : 접근 제어가 목적
	==데코레이터 패턴== : 새로운 기능 추가가 목적

| 기준    | 데코레이터 패턴                           | 프록시 패턴                                            |
| ----- | ---------------------------------- | ------------------------------------------------- |
| 목적    | 객체에 동적으로 새로운 기능을 추가하기 위함           | 다른 객체에 대한 접근을 제어하거나 부가기능을 제공하기 위함                 |
| 사용 시점 | 실행 시간에 객체의 기능을 확장하고자 할 때           | 객체의 생성이나 접근에 대한 제어가 필요할 때                         |
| 구현 방식 | 추상 클래스나 인터페이스를 사용하여 객체에 새로운 기능을 추가 | 프록시 객체를 통해 실제 객체에 접근, 프록시 객체와 실제 객체는 같은 인터페이스를 구현 |

### 데코레이터 패턴
1.객체들이 사용하는 코드를 훼손하지 않으면서 런타임에 추가 행동들을 객체들에 할당할 때 사용.
- 상속을 사용하여 객체의 행동을 호가장하는 것이 어색하거나, 불가능할 때 사용할 수 있다.
	-만일 final 키워드가 기입된 클래스의 경우는 데코레이터 패턴을 통해 래핑하여 재사용 할 수 있다.

ex)
==택스트 편집기==
	- 택스트 편집기에서 굵게, 이텔릭체, 밑줄 등과 같은 다양한 텍스트 포맷을 지원한다.
==Spring==
	- HttpServletRequestWrapper : Sevlet에서 제공하는 Wrapper로 데코레이터 패턴을 지원한다.


### 프록시 패턴
4. 가상 프록시, 지연 로딩이 필요한 경우
	- 부담되는 서비스 객체를 바로 초기화 한다면 리소스 낭비가 발생 할 수 있으므로, 프록시 객체를 통해 객체를 초기화 할 수 있다.

 1. 보호 프록시, 접근 제어가 필요한 경우
	- 특정 클라이언트에 대해서만 서비스 객체를 이용할 수 있도록 하려는 경우 프록시 객체를 통해서 처리할 수 있다.

5. 원격 프록시, 원격 서비스의 로컬 실행이 필요한 경우
	- 서비스 객체가 원격 서버에 있는 경우에는 네트워크를 통해 클라이언트의 요청을 전달하여 처리할 수 있다.

6. 로깅 프록시, 서비스 객체에 대한 로깅이 필요한 경우
	- 프록시 객체에서 서비스에 전달하기 전과 후로 로깅을 진행할 수 있다.

7. 캐싱 프록시, 요청 결과를 캐시하고 생명주기를 관리해야 하는 경우

ex) ==Spring JAP==
- JPA의 지연 로딩의 경우, 가상 프록시를 적용하여 실제로 객체를 조회하기 이전까지 프록시 객체로 Entity를 대신하여 제공한다.
ex) ==Spring AOP==
- Spring AOP는 프록시 패턴을 사용하여 트렌젝션 관리, 로깅, 보안 등의 작업을 프록시에서 처리한다.

``` java

public void write(char cbuf[], int off, int len) throws IOException {
    synchronized (lock) {
        ensureOpen();
        if ((off < 0) || (off > cbuf.length) || (len < 0) ||
            ((off + len) > cbuf.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }

        if (len >= nChars) {
            /* If the request length exceeds the size of the output buffer,
                flush the buffer and then write the data directly.  In this
                way buffered streams will cascade harmlessly. */
            flushBuffer();
            out.write(cbuf, off, len);
            return;
        }

        int b = off, t = off + len;
        while (b < t) {
            int d = min(nChars - nextChar, t - b);
            System.arraycopy(cbuf, b, cb, nextChar, d);
            b += d;
            nextChar += d;
            if (nextChar >= nChars)
                flushBuffer();
        }
    }
}

```

---
# Spring의 지연로딩?
- JPA에서 ==실제 데이터가 필요한 시점까지 데이터베이스 조회를 지연하는 기법==
- 엔티티를 처음 조회할 때는 연관된 데이터를 즉시 로드하지 않고, 그 연관된 데이터가 실제로 사용될 때 데이터베이스에서 조회하는 방식
- ==불필요한 데이터 조회를 줄여서 성능을 최적화하는데 유리하다.==

## 지연로딩(Lazy Loading)의 기본동작.
- 지연 로딩을 설정하면 연관된 엔티티나 컬렉션은 처음에 프록시 객체로 로드된다.
	- 프록시는 실제 엔티티를 대신하는 객체로, DB조회가 필요할 때 프록시가 실제 데이터를 조회하여 값을 제공
		- 처음 부터 연관된 데이터를 모두 로드하는것이 아닌, 실제 접근 시점에 DB에서 로드되도록 지연

#### 자신과 연관된 엔티티를 실제로 사용할 때 연관된 엔티티를 조회하는 방식
- 엔티티 A를 조회시 관련(Reference)되어 있는 엔티티 B를 한 번에 가져오지 않는다.
- 프록시를 맵핑하고 실제 B를 조회할 때 쿼리가 나간다.
*쿼리가 총 두 번 나간다. A조회시 한 번, B조회시 한 번*

---
# Java의 리플렉션이란?
- 구체적인 Class Type을 알지 못하더라도 해당 Class의 method, type, variable들에 접근할 수 있도록 해주는 JAVA의 API이다.
- 컴파일된 바이트 코드를 통해 Runtime에 동적으로 특정 Class의 정보를 추출할 수 있는 프로그래밍 기법

## '바인딩'이란?
- 프로그램에 사용된 구성 요소의 실제 값 또는 프로퍼티를 결정짓는 행위
	-즉 프로그램에서 사용되는 변수나 메서드 등 모든 것들이 결정되도록 연결해주는 것

### 정적 바인딩 
	- 컴파일 시점에 결정
	- 프로그램이 실행 돼도 변하지 않음
	- 오버로딩 : 컴파일 다형성, 메서드 타입,개수,순서 를 다르게 하여 정의하는 것
	- private, final, static이 붙은 메서드

### 동적 바인딩
	- 런타임 시점에 결정
	- 오버라이딩 : 런타임 다형성, 부모,상위 클래스의 메서드를 하위 클래스가 재정의하여 사용하는 것
	- Java에서의 다형성, 상속이 가능한 이유

## Reflection 사용 이유
- 클래스의 수정 없이 유연하게 확장 가능한 코드를 작성할 수 있다.

- 앞서 설명했던 것을 토대로 생각해보면, Reflection은 Runtime에 Class Type을 모르는채로 객체를 생성하고 이용하기 때문에 동적 바인딩을 제공한다.

### 동적으로 Class를 사용 해야할 경우
- 코드 작성 시점에서는 어떠한 Class를 사용해야할지 모르지만 Runtime에 Class를 가져와서 실행해야하는 경우 (Spring Annotation)

### Test Code 작성
- private 변수를 변경하고 싶거나 private method를 테스트할 경우

### 자동 Mapping 기능 구현
- IDE 사용 시 Da 입력만해도 이와 관련된 Class 혹은 Method 목록들을 IDE가 먼저 확인하고 사용자에게 제공한다


## Reflection 사용 방법
- Reflection은 아래와 같은 정보를 가져올 수 있다.
8. Class/Interface
9. Constructor
10. Method
11. Field

해당 정보들을 통해 (1) 객체 생성 (2) 메서드 호출 (3) 변수 값을 변경할 수 있다.

[1] Class / Interface
``` java
public static void main(String[] args) throws Exception {
	// 1. class를 알고 있을 경우
    Class car = Car.class;
    
    // 2. class 이름만 알고 있을 경우
    Class car = Class.forName("com.reflection.test.Car");
    // class.getName() -> com.reflection.test.Car
 
    // 3. Default 생성자를 이용한 객체 생성
    Car realCar = car.newInstance();
    
    // 4. class에 구현된 interface 확인
    Class[] interfaces = car.getInterfaces();
}
```
[2] Constructor
``` java
public static void main(String[] args) throws Exception {
    Class car = Class.forName("com.reflection.test.Car");
    
    // 1. 인자가 없는 생성자 가져오기
    Constructor constructor = car.getDeclaredConstructor();
    
    // 2. String 인자를 가진 생성자 가져오기
    Constructor constructor = car.getDeclaredConstructor(String.class);
    
    // 3. 모든 생성자 가져오기
    Constructor constructors[] = car.getDeclaredConstructors();
    
    // 4. public 생성자만 가져오기
    Constructor constructors[] = car.getConstructors();
    // public com.reflection.test.Car()
	// public com.reflection.test.Car(java.lang.String)
    
    // 5. 생성자를 이용한 객체 생성
    Car realCar = constructor.newInstance();
}
```
[3] Method
``` java
public static void main(String[] args) throws Exception {
    Class car = Class.forName("com.reflection.test.Car");
  
    // 1. 인자가 없는 method 가져오기
    Method method = car.getDeclaredMethod("move");
    
    // 2. String 인자를 가진 method 가져오기
    Method method = car.getDeclaredMethod("move", String.class);
    
    // 3. 모든 method 가져오기
    Method methods[] = car.getDeclaredMethods();
    
    // 4. 상속받은 method와 public method 가져오기
    Method methods[] = car.getMethods();
	// public void com.reflection.test.Car.move()
	// public void com.reflection.test.Car.move(java.lang.String)
    
    // 5. method 호출
    Class realCar = car.newInstance();
    method.invoke(realCar, /*인자*/);
    
    // 6. 접근 제한자를 무시한 method 호출.
    method.setAccessible(true);
    method.invoke(realCar, /*인자*/);
}
```
[4] Field
``` java
public static void main(String[] args) throws Exception {
    Class car = Class.forName("com.reflection.test.Car");
    
    // 1. car 객체에서 name 에 해당하는 field 가져오기
    Field field = car.getDeclaredField("name");
    
    // 2. car + car super 객체를 포함하여 name에 해당하는 field 가져오기
    Field field = car.getField("name");
    
    // 3. car 객체에 선언된 모든 field 가져오기
    Field[] fields = car.getDeclaredFields();
    // private java.lang.String com.reflection.test.Car.name
	// public java.lang.Integer com.reflection.test.Car.type
    
    // 4. car + car super 객체의 모든 public field 가져오기
    Field[] fields = car.getFields();
	// public java.lang.Integer com.reflection.test.Car.age
}
```
[5] Field 값 변경
``` java
public static void main(String[] args) throws Exception {
    Class class = Class.forName("com.reflection.test.Car");
    Constructor constructor = class.getConstructor()
    Car car = constructor.newInstance()
        
    Field field = car.getField("name");
    
    // 1. public field 일 경우
    field.set(car, "아반떼");
    
    // 2. private field 일 경우
    field.setAccessible(true);
    field.set(car, "아반떼");
}
```
---
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

# Static 키워드에 대하여
- 클래스 레벨의 변수나, 메서드, 블록 을 정의 할 때 사용된다.
- 인스턴스(객체)생성 , ==new키워드 없이도 접근이 가능==하다.
- ==모든 인스턴스에서 공유==된다.
- static변수는 프로그램이 빌드될 때 메모리에 할당, 종료될 때 까지 유지.
*static키워드의 남용은 OOP의 원칙과 상반되며, 메모리 사용량의 증가로 이어질 수 있으므로 주의가 필요하다.*

## static변수
- 클래스 레벨에서 공유되는 값을 정의할 때 사용.
- 인스턴스마다 별도의 복사본을 유지 할 필요가 없다.
- 메모리 사용을 최적화 할 수 있다.

## static메서드
- 클래스 생성없이 직접 호출 가능.
- 유틸리티 함수나, 상태가 필요없는 연산에 주로 활용. ex) 수학 연산
- 인스턴스 생성의 오버헤드 없이 빠른 실행이 가능하다.
*단, static메서드는 해당 클래스 내의 다른 static메서드나, 변수에만 접근 할 수 있으므로 인스턴스 멤버에 접근해야하는 경우 사용할 수 없다.*

#### 즉, Static은 코드의 재사용성과 효율성을 높일 수 있지만, 객체지향 원칙과 메모리 관리를 고려하여 옳바르게 사용하도록 해야한다.

---

# JVM의 Stack과 Heap

![[Pasted image 20250122111223.png]]
## 스택(Stack)영역
- int, long, booolean 등 기본 자료형을 생성할 때 저장하는 공간
- ==임시적으로 사용되는 변수나 정보들이 저장되는 영역==

메서드 호출 시 마다 ==스텍 프레임(그 메서드만을 위한 공간==이 생성되고, 그 메서드 안에서
사용되는 값들을 저장하고, 호출된 메서드의 매개변수, 지역변수, 리턴 값 등을 임시로 저장한다.

그리고 메서드의 수행이 끝나면 프레임별로 삭제된다.

*단, 데이터 타입에 따라 스텍과 힙에 저장되는 방식이 다르다는 점을 유의해야한다.*
#### 1. 기본(원시)타입 변수는 스텍영역에 직접 값으로 저장된다.
#### 2. 참조타입(new연산자) 등은 힙영역이나 메소드 영역의 객체 주소를 가진다.
*위사진 참고*

### 스텍 영역은 각 스레드 마다 하나씩 존재한다.
### 스레드가 시작될 때 할당되며, 고정된 사이즈를 갖는다.
	- 이를 넘어서면 StackOverFlowError를 발생한다.

## 힙(Heap)영역
- 메서드 영역과 함께 ==모든 스레드가 공유==한다.
- JVM이 관리하는 프로그램 상에서 데이터를 저장하기 위해 런타임 시 ==동적으로 할당되어 사용==되는 영역이다.
- ==new연산자로 생성되는 클래스와 인스턴스 변수, 배열 타입 등 Reference Type이 저장된다.==
- 메서드 영역에 저장된 클래스만이 힙영역에 생성이 되어 적재된다.
*힙영역에 더이상 아무도 참조하지않는 객체가 있다면, GC에 의해 제거된다.*

---