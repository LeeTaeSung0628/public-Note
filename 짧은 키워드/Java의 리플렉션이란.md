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