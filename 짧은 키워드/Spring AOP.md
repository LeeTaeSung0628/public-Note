# Spring AOP

## AOP위빙 방식

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
