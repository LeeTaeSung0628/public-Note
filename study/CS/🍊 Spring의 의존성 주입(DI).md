# 🍊 Spring의 의존성 주입(DI)

#공부 #SPRING #FRAMWORK #DI #의존성주입

---
# Spring에서의 의존성 주입 방법

## Filed Injection(필드 주입)

``` java
	@Component
	public class Controller{
		@Autowired
		private Service servic
	
	...
	}
```

### 사용하면 안되는 이유 
	- 단일책임의 원칙 위반하기 쉬워진다.
	- 의존성을 주입하기 쉽기 때문에, @Autoqwired아래에 개수 제한없이 추가할 수 있다.
	- 이때, 하나의 class가 많은 책임을 갖고, 순환참조가 이뤄질 수도 있기 때문에
	의존성이 높아져, 사용을 피하는것이 좋다.

----------------------

## Setter Injection(수정자 주입)

``` java
	@Component
	public class Controller{
		private Service servic
	
		@Autowired
		private void setService(Service servic){
			this.service = service;
		}
	}
```

선택적인 의존성을 사용할때 유용하다.
스프링 3.x 에서는 수정자 주입을 권장하고있다.

### 문제점
	- 수정자 주입을 사용하게 되면, service구현제를 주입하지 않아도 
	controller객체는 생서이 가능하기때문에 널포인터익셉션이 발생할 가능성이 있다.
	주입이 필요한 객체가 주입되지 않아도 얼마든지 객체를 생성할 수 있다는것이 문제다.

----------------------

## Contructor Injection(생성자 주입)

``` java
	@Component
	public class Service{
		private DAO dao;
	
		//생성자
		@Autowired //생략가능
		public Service(DAO dao){
			this.dao = dao;
		}
	}
	
	@Component
	public class Controller{
		private final Service servic = new Service( new DAO( ) );


		/* Spring사용시 */
		
		private final Service service;
			
		// 생성자 주입
		@Autowired
		public Controller(Service service) {
			this.service = service;
		}

	}
```

### 생성자 주입을 사용해야 하는 이유
- Spring Framework 에서 권장하는 방법으로, **필수적으로 사용해야하는 의존성 없이는 객체를 만들지 못하도록 강제**할 수 있기 때문에 사용을 권장하고 있다.
- Spring 4.3 버전 이후부터는 Class를 완벽하게 DI Framework로 부터 분리할 수 있다.
- 단일 생성자에 한해서 @Autowired를 붙이지 않아도 된다.
- 필드 주입과 수정자 주입은 final로 선언할 수 없지만, 생성자 주입은 
final로 필드 객체를 선언하여 런타임에 불변성을 보장한다.
- 앞서 설명한 필드 주입의 모든 단점을 보완할 수 있다.

#### 생성자 주입의 경우에도 순환참조가 똑같이 일어날 수 있지만, 필드, 수정자 주입과 다르게 ==런타임== 시가 아니라, ==컴파일== 시 미연에 찾아낼 수 있다.

