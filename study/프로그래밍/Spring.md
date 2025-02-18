
# 🍃Spring

#공부 #SPRING #FRAMWORK

---

# @ 어노테이션이란 무엇인인가?
	1. 컴파일러에게 코드작성 문법에러를 체크하도록 정보 제공
	2. 빌드나 배치시 코드를 자동으로 생성할수 있도록 정보 제공
	3. 실행시 특정 기능을 수행하도록 정보 제공


---
# Bean이란 무엇인가?
	- Spring컨테이너에 의해 관리되는 재사용 가능한 컴포넌트이다.
	즉 스프링이 관리하는 자바 객체 이다.
	- getter/setter를 포함한다.


---
# @Component는 내가 만든 클래스에 Bean을 주입하는역할을 한다.
	이때 객체가 생성된다면 싱글톤 패턴으로 생성이 된다.

	** Spring Framework의 도움을 받으면 단점을 줄이고 장점을 사용할 수 있다.
**@Controller / @Service / @Repository 등의 어노테이션을 포함한다.
*싱글톤패턴?
- 인스턴스(객체)가 오직 한개만 생성된다
- 장점 : 메모리낭비를 방지한다.
	이미 생성된 객체를 재사용하기 때문에 속도가 빠르며
	전역으로 사용되는 인스턴스이기 때문에 데이터공유가 쉽다.
- 단점 : 테스트에 어려움이있다.
	자식클래스를 만들수없다.
	내부의 상태를 변경하기 어렵다.
 
 
### 이때, @Component와 @Controller의 차이점은 무엇일까?
	 - 컴포넌트와 컨트롤러 모두 Bean객체를 등록하지만
	컨트롤러는 url과 클래스를 이어주는 역할을 할 수 있다.


---

#  Spring 어노테이션@ 설명

## @RequestBody는 API호출시 넘겨주는 파라미터값(JSON등)을
	JAVA오브젝트로 자동 변환해준다. (Controller기준 받는 입장)
- 프론트앤드에서 Ajax요청시 JSON형식으로 값이 넘어오는데,
 이 JSON형식을 받을때 사용하는 어노테이션이다. 
- 자동으로 자바객체로 바꿔준다.

## @RequestParam은 외부API호출시 넘겨주는 파라미터값을 가져옴
	- 외부API에서 name으로 넘긴 값을 String name에 저장함.

## @Autowired는 빈(의존성)을 주입받기위해 사용.
	- Autowired/생성자/setter 이렇게 총 3가지 방법으로 의존성을 주입받을 수 있다.

## @Entity는 데이터베이스와 직접 연결된 클래스를 설정한다.

## @PathVariable은 URL(경로)에서 변수를 추출하여 매개변수에 할당한다.
	- 경로변수는 {id}로 둘러싸인 값을 의미한다.
	- 주로 상세조회, 수정, 삭제 등의 작업에서 리소스 식별자로 사용된다.

## @Generated는 개발자가 아닌, DB에서 기계적으로 생성한 클래스이다. 
	하이버네이트가 오브젝트를 대신 갱신한다.

## @Builder는 빌더패턴을 완벽하게 지원해주는 어노테이션이다.
	*빌더 패턴이란? : 생성자에 파라미터를 주입하여 생성하는것이 아닌, 별도의 Builder를 두어서 객체를 생성하는것을 말한다.
	생성자가 없는경우 : 모든 맴버 변수를 파라미터로 받는 기본 생성자 생성
	생성자가 있는 경우 : 따로 생성자를 생성하지 않음

## @Data
	@Getter/@Setter/@ToString/@EqualsAndHashCode/@RequiredArgsConstructor를 자동으로 적용시켜준다.

## @RequiredArgsConstructor
	@NonNull이나 final이 붙은 필드값 들에 대해 생성자를 자동으로 생성해준다. ( @Autowired를 사용하지 않고 의존성주입이 가능하다)
	해당 어노테이션을 사용하면 클래스가 의존하고 있는 필드를 간단하게 초기화할 수 있다.

## @AllArgsConstructor 는 모든 필드값을 파라미터로 받는 생성자를 생성해준다.
	해당 어노테이션을 사용하면 클래스의 모든 필드값을 한 번에 초기화할 수 있다.

## @NoArgsConstructor 는 파라미터가 없는 디폴트 생성자를 생성해준다.
	해당 어노테이션을 사용하면 클래스에 명시적으로 선언된 생성자가 없더라도 인스턴스를 생성할 수 있다.

	개발자가 실수로 클래스의 필드 중 하나의 필드에 대한값 설정을 누락 시킬수도 있어, 객체는 불완전한 상태가 되어버린다.
	이를 방지하고자 모든 필드값을 가지도록 강제하고 싶다면, AccessLevel.PROTECTED 속성을 부여해줘 해결할 수 있다.
	다음과 같은 속성을 부여해주면, 기본 생성자의 접근 제어가되어 IDE단계에서 누락을 방지할 수 있다.
## @EqualsAndHashCode는 equals와 hashCode를 자동으로 생성해준다.
	- equals는 두 객체의 내용이 같은지, 동등성을 비교하는 연산자이다.
	- hashcode는 두 객체가 같은 객체인지, 동일성을 비교하는 연산자 이다.


#### 빌더패던의 장점
1. 생성자의 파라미터가 많은 경우 가독성이 떨어진다.
	- 빌더패턴으로 생성하는 경우 각 값들이 함수로 셋팅이되고, 각각 무슨값들이 어떠한 것을을 의미하는지 파악하기가 수월하다.
	ex)
	Bag bag = new Bag("name", 1000, "memo", "20", "30");
		vs
	Bag bag = Bag.builder()
		,name("name")
		,money(1000)
		,memo("memo")
		,won(20)
		,dolor(30)
		,build();
2. 어떠한 값을 먼저 넣더라도 상관없다(순서x)
	- 생성자의 경우 정해진 파라미터대로 값을 입력해야 정해진 값에 매핑이 되지만, 빌더패던의 경우 필드 이름을 기준으로 값을 삽입하게 때문에 순서를 생각하지 않아도 된다.
#### 빌더패턴 구현
- @NoArgsConstructor로 기본 생성자의 생성을 방지하고, @Builder를 이용하여 객체의 생성에 유연성을 준다.
이때, 이 2개의 어노테이션을 함께 사용하기 위해서는 @AllArgsConstructor 어노테이션이 필요하다.

이유 :  @Build는 위에서 설명한 바와 같이 생성자가 없다면 모든 파라미터를 갖는 생성자를 생성하지만, @NoArgsConstructor로 인해 아무런 생성자를 생성하지 않는다. 이때 build메서드를 사용하여 모든파라미터를 받는 메서드(생성자)를 동작시키면 매칭되는 생성자가 없기때문에 오류를 야기한다.
따라서 @AllArgsConstructor 어노테이션을 추가로 작성하여 해결할 수 있다.

더 깔끔한 방법으로는 직접 생성자를 생성해주고, 빌더 패턴에서 해당 생성자를 사용하도록 하는 방법도 있다.


---
# Lombok 사용시 주의사항

## 1. @AllArgsConstructor / @RequiredArgsConstructor 사용시
	- 위 두개의 어노테이션을 편리하게 생성자를 자동으로 생성해 주지만, 주의를 요할 필요가 있다.

- 어떠한 클래스에서 순서대로 인자를 받는 생성자가 있다고 했을 때,  개발자가 임의로 순서를 변경할 경우, 리펙토링은 전혀 작동하지 않고, lombok이 개발자가 인지하지 못하는 사이에 순서에 맞춰 두 필드를 변경해 버린다.
- 그렇기 때문에 순서의 구애받지 않는 @Builder 어노테이션을 사용한는 것을 권장하고 있다.

## 2. 무분별한 @EqualsAndHashCode 사용
	 Mutable(변경가능한)객체에 아무런 파라미터 없이 그냥 사용하는 경우에 문제가 발생할 수 있다.

- 동일한 객체임에도 불구하고 Set으로 필드값을 변경하게 되면, hashCode가 변경되면서 찾을 수 없게되는 부분이 있다. 

## 3. @Data 사용금지
	- 위에서 설명한 @RequiredArgsConstructor 및 @EqualsAndHashCode를 포함하고 있기 때문에 사용을 피하는 것이 좋다.

## 4. @Value 사용 금지
	- 불변 클래스를 생성해주는 @Value또한, @EqualsAndHashCode와 @AllArgsConstructor를 포함하고 있기 때문에 사용을 피하는것이 좋다. 
	- 불변클래스 이기 때문에 @EqualsAndHashCode는 문제가 되지 않지만, AllArgsConstructor가 문제를 일으킬 가능성이 있다.


---
# SessionStorage 와 LocalStorage
	- 두가지 모두 브라우저 저장 장소이다.
	주로, 휘발성 데이터를 저장할 목적을 갖고있다. 


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
	- 단일책임의 원칙 위반
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
		@Autowired
		private Service(DAO dao){
			this.dao = dao;
		}
	}
	
	@Component
	public class Controller{
		private final Service servic = new Service( new DAO( ) );
	}
```

### 생성자 주입을 사용해야 하는 이유
	- Spring Framework 에서 권장하는 방법으로, 
	필수적으로 사용해야하는 의존성 없이는 객체를 만들지 못하도록 강제할 수 있기
	때문에 사용을 권장하고 있다.
	- Spring 4.3 버전 이후부터는 Class를 완벽하게 DI Framework로 부터
	분리할 수 있다.
	- 단일 생성자에 한해서 @Autowired를 붙이지 않아도 된다.
	- 필드 주입과 수정자 주입은 final로 선언할 수 없지만, 생성자 주입은 
	final로 필드 객체를 선언하여 런타임에 불변성을 보장한다.
	- 앞서 설명한 필드 주입의 모든 단점을 보완할 수 있다.

#### 생성자 주입의 경우에도 순환참조가 똑같이 일어날 수 있지만, 필드, 수정자 주입과 다르게 ==런타임== 시가 아니라, ==컴파일== 시 미연에 찾아낼 수 있다.

하지만 실무에서는 필드 주입을 주로 사용한다.
이유 : 가장 구현하기 쉽고, 읽기 편하기 때문이다.


---
# 데이터 검증 (Validation)

Spring에서의 데이터 검증(Validation)은 여러 계층에 거쳐서 발생한다.
여기서, 가장 기본적인 검증 방법은 Bean Validation이다.
- '필드'에 특정 어노테이션을 적용하여 필드가 갖는 제약 조건을 정의하는 구조로 이루어진 검사다.
validator가 그 클래스로 생성된 객체의 유효성 여부를 확인한다.
이때, 어떠한 비즈니스로직에 대한 검증이 아닌, 객체 자체 필드에 대한 검증을 한다.

``` java
	@RestController
	@AllArgsConstructor
	public class BookController {
	    private BookService bookService;
	
	    @PostMapping("/books")
	    public void save(@RequestBody @Valid AddBookRequestDto addBookRequestDto, BindingResult bindingResult) {
	        if(bindingResult.hasErrors()) {
	            bindingResult.getAllErrors()
	                .forEach(objectError->{
	                    System.err.println("code : " + objectError.getCode());
	                    System.err.println("defaultMessage : " + objectError.getDefaultMessage());
	                    System.err.println("objectName : " + objectError.getObjectName());
	                });
	            return;
	        }
	        bookService.save(addBookRequestDto.toEntity());
	    }
```

-> 여기서@Valid 어노테이션이 Request에 있는 방인된 객체(dto)의 유효성을 확인하고
 유효하지 않은객체라면 BindingResult 파라미터에 들어가게 된다.


그렇다면, @PathVariable과 @RequestParam은 어떻게 유효성 검사를 진행할까?
- 클래스에 @Validated 어노테이션을 등록해 주면 된다.

	@RestController
	@Validated
	public class UserController {
	
	}


---

# Entity와 DTO를 분리해야하는 이유

	- Entity와 관련된 코드들은 많은데 비해 Dto의 경우는 상대적으로 적다.
	그런 상황에서 Entity의 변경가능성은 Dto의 비해서 또 적다.
	만약 Entity를 Request나 Respons에 사용하게 된다면 변경 가능성이 높아지고,
	동시에 같이 변경되는 코드들이 늘어나기 때문에 코드 유지보수를 생각했을때 분리하는것이 옳다.


---
# Spring의 EntityManager는 무엇일까?

	- 엔티티 매니저는 DB커넥션처럼 사용된다. 즉, 엔티티매니저를 절대 공유해서는 안된다.
	하나의 스레드에서만 사용해야하며, 사용이 끝나면 반납해야한다.
	그 이유는 트렌젝션 단위로 엔티티매니저를 사용하기 때문이다.
	여러 쓰레드가 동시에 사용하게 된다면 영속성 과 DB간의 데이터동기화가 깨지게 되기 때문이다.


---
# DB의 저장 프로시저 (SP : Stored Procedure)란?

	- 저장 프로시저는 각 DBMS에서 제공하는 기능으로, 
	SQL문을 저장해놓고, 필요할 때마다 호출해서 사용하는 프로그래밍 기능이다.

### 사용하는 이유
1. SQL의성능을 향상시킬수 있다.
	- SP를 실행하게 되면 최적화, 컴파일 단계를 거쳐, 결과가 캐시에 저장되게 되는데, 
	이 후 해당 SP를 실행하게 되면 캐시에 있는것을 가져와 사용하므로 실행속도가 빠르다.

2. 유지보수 및 재활용 측면에서 유리하다.
	- 응용프로그램 내에서 직접 SQL문을 호출하지않고 SP이름을 호출하도록 설정하면
	SP파일만 수정하면 되기때문에 유지보수와 재활용 측면에서 유리하다

3. 보안이 강화될 수 있다.
	- 사용자별로 테이블 권한을 부여하는것이 아닌, SP에만 접근 권한을 주는 방식으로 보안을 강화할 수 있다.
	실제 테이블에 접근하여 조작하는것이 위험하기 때문에 개발자에게는 SP권한만 주는 방식을 많이 사용한다.

또한, 일반적인 쿼리들은 Where의 조건이 조금만 달라져도, 최적화 컴파일을 다시 수행하여야 하지만,
함수 형태의 SP로 생성하게 되면 매개변수만 변경하여 성능적인 측면을 크게 높일 수 있다.


---
#  Spring에서의 저장 프로시저 동작원리

	1. DB에서 SP를 생성한다( 미리 작성되어있는 쿼리 모음 )
	2. SP의 리턴값을 저장하는 Entity클래스(@NamedStoredProcedureQuery어노테이션)으로 연결함.
	3. Repository에서 프로시저 객체를 생성한다. 이때, 2.에서 생성한 프로시저JPA의 파라미터를 설정한 후 execute한다.


궁금한점 : 저장 프로시저를 사용할경우 Entity는 테이블명으로 class와 연결하지 않아도 괜찮은것인가?
- @NamedStoredProcedureQuery어노테이션을 Entity에 적용하면 DBMS에 정의 되어있는 SP(저장 프로시저)
와 연동하여 사용할 수 있다.


---
# JPQL과 QueryDSL 비교

## JPQL 
``` java
	String username = "java";
	String jpql = "select m from Member m where m.username = :username";
	List<Member> result = em.createQuery(query, Member.class).getResultList()
```

jpql이란 JPA의 일부로, 쿼리를 Table이 아닌 객체 기준으로 작성하는 객체지향 쿼리 언어 이다.

문제점 : String형태 이기 때문에 개발자 의존적인 형태를 띈다.
	컴파일 단계에서 Type-Check가 불가능하다.
	런타임 단계에서 오류가 발생한다.(장애 리스크가 증가한다)

-----------------

## QueryDSL
``` java
	String username = "java";
	
	List<Member> result = queryFactory
        .select(member)
        .from(member)
        .where(usernameEq(username))
        .fetch();
```

QueryDSL은 해당 문제를 해결하기 위해서 나온 기능이다.

장점 : 문자가 아닌 코드로 쿼리를 작성할 수 있어 컴파일 시점에 문법오류 확인 가능.
       IDE의 자동완성 기능의 도움을 받을 수 있다.
       복잡한 쿼리나 동적 쿼리 작성이 편하다.
       쿼리 작성시 제약조건 등 메서드를 추출해서 재사용할 수 있다.



---

# JPA에서 BooleanBuilder 와 BooleanExpression 이란?
	- 두 클래스 모두 JPA에서 동적쿼리를 제작할 때 사용한다.
	BooleanBuilder는 if문을 각 데이터 조건에 맞게 코드로 작성할 수 있도록 도와주는 역할을 한다.

BooleanExpression 는 메서드를 생성하여 where절 안에서 호출하여 구현할 수 있다.
메서드 안에서 where절로 null이 반환되면 해당 조건이 무시되기 때문에 동적쿼리가 가능하다.
(모든 조건이 NULL을 반환하면 전체 엔티티를 불러오는 점을 주의하자)
메서드는 재사용이 가능하고, 메서드들 끼리 재조합도 가능하기 때문에 유지보수나, 재활용에 유리하다.

*동적쿼리 - 실행시점에서, 사용자나 프로그램의 사정에따라 쿼리의 조건이나 구조를 동적으로 결정할때 사용.


---
# JPAQueryFactory란?
	- SQL문법과 유사하게 복잡한 쿼리를 제작할 수 있게 도와주는 SPRING의 JPA지원 클래스이다.

ex)
``` java
    private RemainDto firstList(DashBoardModel model) {
        return factory
                .select(
                        Projections.fields(
                                RemainDto.class,
                                cfProductInvest.amount.sum().coalesce(0L).as("total"),
                                new CaseBuilder()
                                        .when(cfProduct.category.eq("2"))
                                        .then(cfProductInvest.amount)
                                        .otherwise(0L).sum().coalesce(0L).as("estate"),
                                new CaseBuilder()
                                        .when(cfProduct.category.ne("2"))
                                        .then(cfProductInvest.amount)
                                        .otherwise(0L).sum().coalesce(0L).as("noneEstate")
                        )
                )
                .from(cfProductInvest)
                .leftJoin(cfProduct)
                .on(cfProductInvest.productIdx.eq(cfProduct.idx))
                .leftJoin(g5Member)
                .on(cfProductInvest.memberIdx.eq(g5Member.mbNo))
                .where(
                        g5Member.mbId.eq(model.getMbId()),
                        cfProductInvest.investState.eq("Y")
                )
                .fetchOne();
    }
```


---

# Spring Batch란?
	- 스프링배치는 엔터프라이즈 시스템의 강력한 배치 어플리케이션을 개발할 수 있도록 설계된 배치프레임워크이다.
	- 일괄처리(Batch Processing), 분산처리 작업을 효율적으로 처리할 수있는 기능 제공.
	- 로깅/추적, 트렌잭션관리, 작업 처리통계, 작업재시작, 리소스관리 등 대용량 레코드 처리에 필수적인 기능을 제공
	- SpringBatch는 JobRepository로 동작하는데, 여기에 Job / JobLauncher / Step이 포함되어있다.

* SpringBatch는 대량의 데이터를 일괄적으로 처리할 뿐 
 특정 주기마다 자동으로 돌아가는 스케줄링 기능은 없다.
 단지, 스케줄러와 함계 사용할 수 있도록 설계되어있을 뿐이다.

그렇기 때문에 스케줄링 라이브러리인 Quartz라이브러리를 추가하여 같이 사용한다.

---

# Batch 용어 설명
### Job 
- 독립적으로 실행할 수 있는 고유하며 순서가 지정된 스텝의 목록
- 애플리케이션 실행시 Job으로 인식되는 bean들이 자동으로 실행된다.
- 1개 이상의 Step을 포함하여 원하는 동작을 실행시킬 수 있다
- 배치 처리 과정 중 전체 계층의 최상단에 위치도

### Step
- job의 구성요소로 자체적인 입력/출력/처리를 가질 수 있다.
- ==tasklet 또는 Chunk==기반 처리를 포함하여 step안에서 수행될 기능들을 명시할 수 있다.
- 트렌젝션은 step내에서 이루어진다. 때문에, 독립되도록 의도적으로 설계된 것이다.

### tasklet
- Step의 작업 단위를 Tasklet으로 정의
- 주로 간단한 작업(단일 데이터 처리, 파일 삭제 등)에 적합하다.

``` java
@Component
public class SimpleTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        System.out.println("Tasklet 방식으로 작업 수행");
        return RepeatStatus.FINISHED;
    }
}

@Bean
public Step step1(StepBuilderFactory stepBuilderFactory) {
    return stepBuilderFactory.get("step1")
        .tasklet(new SimpleTasklet())
        .build();
}

@Bean
public Job job(JobBuilderFactory jobBuilderFactory, Step step1) {
    return jobBuilderFactory.get("job")
        .start(step1)
        .build();
}

```


### Chunk
- 대량 데이터를 일정 크기(chunk)로 나누어 처리한다.
- Reader / Processor / Writer로 구성된다.
``` java
@Bean
public FlatFileItemReader<String> reader() {
    return new FlatFileItemReaderBuilder<String>()
        .name("fileReader")
        .resource(new ClassPathResource("input.txt"))
        .lineMapper(new DefaultLineMapper<String>() {
            {
                setLineTokenizer(new DelimitedLineTokenizer());
                setFieldSetMapper(new PassThroughFieldSetMapper());
            }
        })
        .build();
}

@Bean
public ItemProcessor<String, String> processor() {
    return item -> "Processed " + item;
}

@Bean
public FlatFileItemWriter<String> writer() {
    return new FlatFileItemWriterBuilder<String>()
        .name("fileWriter")
        .resource(new FileSystemResource("output.txt"))
        .lineAggregator(new PassThroughLineAggregator<>())
        .build();
}

@Bean
public Step step(StepBuilderFactory stepBuilderFactory) {
    return stepBuilderFactory.get("step")
        .<String, String>chunk(10)
        .reader(reader())
        .processor(processor())
        .writer(writer())
        .build();
}

@Bean
public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
    return jobBuilderFactory.get("job")
        .start(step)
        .build();
}

```

### Job Listener
- 잡 리스너를 이용해서 스프링batch 생명주기의 여러로직을 추가할 수 있다.
ex) beforeJob , afterJob 등등


# hfbatJobScheduler(스케줄러)가 Job으로 등록되어있는 녀석들을 순차적으로 실행될할 때 , 누가 트리거 역할을 하는지
=> 스케줄러는 Jenkins에서 SSH스크립트를 통해 주기적으로 실행한다. Controller에 POST주소가 맵핑되어있는 이유는 테스트로 직접 실행하기 위함이다.

---

# Spring Batch에서의 파티셔닝(Partitioning)
- Tasklet 방식과 Chunk 방식 모두 사용 가능하지만, 보통 Chunk방식에서 사용한다.
### 파티셔닝(Partitioning)
- 대량 데이터를 효율적으로 처리하기 위해 데이터를 여러 개의 작은 조각(Partition)으로 나눠 **병렬적으로 처리**하는 기술입니다. Spring Batch에서 파티셔닝은 **마스터-슬레이브 패턴**을 사용하며, 마스터는 작업을 분할하고 슬레이브는 각각의 분할된 작업을 수행합니다.

#### 파티션너(파티션 별 범위 - 컨텍스트 범위 설정)
``` java
import org.springframework.batch.core.partition.support.Partitioner;
import org.springframework.batch.item.ExecutionContext;
import java.util.HashMap;
import java.util.Map;

public class RangePartitioner implements Partitioner {

    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        Map<String, ExecutionContext> partitions = new HashMap<>();
        int min = 1; // 데이터베이스 ID의 최소값
        int max = 5; // 데이터베이스 ID의 최대값
        int targetSize = (max - min) / gridSize + 1; // 각 Partition의 범위 크기

        int start = min;
        int end = start + targetSize - 1;

        for (int i = 0; i < gridSize; i++) {
            ExecutionContext context = new ExecutionContext();
            context.putInt("minId", start); // 시작 ID
            context.putInt("maxId", end); // 종료 ID
            partitions.put("partition" + i, context);

            start += targetSize;
            end += targetSize;
        }

        return partitions;
    }
}
```

### **Chunk와 Partitioning의 병렬 실행 예제**

#### 예제 시나리오:

- 데이터베이스에 100개의 레코드가 있음.
- `gridSize = 4`: 데이터를 4개의 Partition으로 나눔.
- `chunkSize = 10`: 각 Partition에서 데이터를 10개씩 읽어 처리.
- 스레드 풀 크기 = 4: 4개의 Partition이 동시에 실행 가능.

---
코드가 여러줄 나올거같진 않아서 방식은 
페어 프로그래밍 방식으로 진행하도록 하죠

개발 절차는 이렇게 갈 예정이에요

4. Chunk, Partioning방식을 Job을 추가 개발 (기존 balanceJob 유지)
5. 당분간 두 Job을 병행하면서 비교
6. 추가한 Job 기능에 문제없다면 기존 balanceJob 삭제

#### 실행 흐름:

7. **Partition 생성**:
    
    - Partition 1: ID 1 ~ 25
    - Partition 2: ID 26 ~ 50
    - Partition 3: ID 51 ~ 75
    - Partition 4: ID 76 ~ 100
8. **각 Partition에서 Chunk 처리**:
    
    - Partition 1:
        - Chunk 1: ID 1 ~ 10 → 커밋
        - Chunk 2: ID 11 ~ 20 → 커밋
        - Chunk 3: ID 21 ~ 25 → 커밋
    - Partition 2:
        - Chunk 1: ID 26 ~ 35 → 커밋
        - Chunk 2: ID 36 ~ 45 → 커밋
        - Chunk 3: ID 46 ~ 50 → 커밋
    - 나머지 Partition도 동일 방식으로 처리.
9. **병렬 실행**:
    
    - 스레드 풀 크기 = 4이므로 4개의 Partition이 동시에 실행됩니다.
    - Partition 처리 순서는 스레드 풀에서 처리되는 순서에 따라 다를 수 있음.
10. **트랜잭션 관리**:
    
    - 각 Partition은 독립적인 트랜잭션을 가짐.
    - 각 Chunk가 커밋될 때마다 트랜잭션이 종료됨.

### ThreadPoolSize : 동시에 실행시킬 스테리드의 개수
### gridSize : 실제로 제단할 사이즈(데이터를 얼마나 세분화해서 각 파티션 작업에 할당할 건지)
### QueueCapacity : 대기열 크기

---


#  APP - SERVICE - API 통신 플로우

투자신청기록쪽 api -> API레스폰스모델 
웹플럭스(비동기 기반)을 사용해서 service프로젝트와 (내부)통신한다.

내부통신을 위해 필요한 헤더, url등을 생성해야하기 때문에 senderutils 클래스를 공통(빈)을 정의해서 만들어 통신.

retrieve ** 중요
웹클라이언트 클래스 객체를 사용해서 uri 콘텐츠 등등 헤더와 바디(데이타)를 정의한다


---
# Spring WebFlux란?
- 스프링5에서 소개된 리엑티브 프로그래밍, 반응형 및 비동기적인 웹 어플리케이션 개발을 지원하는 모듈이다.
![[Pasted image 20240611144129.png]]

*리엑티브 프로그래밍이란 ?
	- 비동기  및 이벤트 기반 애플리케이션을 개발하기 위한 패러다임으로, 주로 높은 확장성과 성능을  제공하는것

내부통신을 이용하여 API프로젝트와 통신할때 주소 맵핑이 어떻게 이루어지는지??
	- Spring WebClient를 이용하여 내부 통신을 한다.
 
## Spring WebClient란??
- SpringWebFlux의 일부로써, 비동기적인 방식으로 HTTP 요청을 보내고 응답을 받을 수 있는 라이브러리이다.
- 웹으로 API를 호출하기 위해 사용되는 HTTP Client모듈 중 하나이다.
- RestTemplate과 같은 기능을 하지만, RestTemplate는 Blocking 방식이고, WebClient는 Non-Blocking방식이다.
Blocking 동기 - Non-Blocking 비동기 ( 정확히 같은것은 아니지만 비슷하다 ? )

- 요청자(APP)에서 WebClient라이브러리를 사용한 senderUtils를 사용하여
프로퍼티 소스와, 송신방식(GET/POST), 넘길 값(DTO), request를 수신할 값(ApiResponseModel)을 설정한다.

#### *Spring MVC환경에서도 SpringWebClient를 사용해도 될까?(현재 프로젝트가 이렇게되있다)
-SpringMVC에서는 WebFlux와 달리, 블로킹I/O를 사용하기 때문에, 동기적인 작업을 수행할 떄에는 WebClient보다 RestTemplat이 효과적이지만, 비동기 작업을 할 때에는 WebClient를 사용하는것이 효과적이다.

---
# Spring WebClient VS RestTemplate

### RestTemplate : Multi-Thread / Blocking방식

- Thread Pool을 애플리케이션 구동시 미리 만들어 두고,
 요청시 가용한 Thread가 있다면 1요청당 1Thread가 할당된다.
- 만약 가용한 스레드가 부족하다면 Queue는 대기하게 되며,
 전체 서비스의 속도가 현저히 느려지게 된다.

### Spring WebClient : Single-Thread / Non-Blocking 방식

- Core당 1개의 Thread를 사용한다.
- 요청은 Event Loop내의 job으로 등록되고, 각 job을 제공자에게 요청한 후,
 기다리지 않고 다른 job을 처리한다.
- Event Loop는 제공자로부터 callback으로 응답이 오면, 그 결과를 요청자에게 제공하낟.
- 따라서 반응성/탄력성/가용성/비동기성 을 보장하기 때문에 동시사용자가 크게 몰렸을때
RestTemplate에 비해 성능이 저하되지 않는다.


---
# DispatcherServlet의 역할
	- 웹 애플리케이션 최전방에서 사용자의 요청을 접수하여 URL기주능로 요청을 처리할 controller를 찾고, 그 controller에 처리를 위임한 후, 결과를 받아서 사용자에게 처리 결과가 담긴 화면을 제공해준다.

- 설정은 web.xml의 정보를 활용한다. 사용자 요청을 처리할 Controller목록과 사용자 에게 보여줄 화면을 찾는 ViewResolver가 있다.

---
# HTTP Method - Mapping

### 1. GetMapping
``` java
#### PathVariable 방법
	@RestController
	public class SecondController {
	
	@GetMapping("/second/{id}") //PK(id)가 (변수)인 페이지를 찾고 싶다
	public String getData(@PathVariable Integer id) {
		return "id : "+id;
	}
#### QueryString 방법
	@GetMapping("/second")
	public String getData2(String title, String content) {
		return "title:"+title+", content :"+content;
	}
```

### 2. PostMapping
``` java
@PostMapping("/second")
	public String postData(String title, String content) {
		return "title:"+title+", content :"+content;
	}
```

##### GetMapping은 PostMapping과 달리 http body값을 받는 메서드가 아니다. 따라서 GET요청에 Body값을 넣으면 null값이 나온다.

##### PostMapping에 body가 아닌 params에 값을 넣어도 값이 정상적으로 출력되는 이유는 params는 GET/POST/PUT/DELETE 모든 값이 나오기 때문이다. PostMapping을 원한다면 Body에 값을 넣어줘야한다

### 3. PutMapping
``` java
	@PutMapping("/second")
	public String putData(String title, String content) {
		return "title:"+title+", content :"+content;
	}
```
- PostMapping과 같은원리로 작동한다

### 4. DeleteMapping

``` java
	@DeleteMapping("/second/{id}")
	//쿼리스트링 해도 됨
	public String deleteData(@PathVariable Integer id) {
		return id+"delete ok";
	}
```
##### DeleteMapping은 요청바디(@RequestBody)를 가지지 않는것이 일반적이다.
##### 그렇기에 @RequestBody를 사용하여 바디를 수신하는 것이 지원되지 않음
- 데이터 전달이 필요한 경우 @RequestParam을 사용하거나
@DeleteMapping 대신에 @PutMapping을 사용하도록 하자

## Get말고는 다들 비슷한 기능같은데 나누는 이유가 무었인지??
	 내생각에는 메서드를 명시적으로 작성 할 수 있기 때문에 더욱 가독성이 높아지는 장점이 있지 않을까 싶다.


---

# Spring Integration이란?
- 애플리케이션 내부-외부 사이의 메시징을 가능하게 하는 프레임워크이다.
- Spring Framwork에서 매세징이란 메타데이터와 함께 결합되어있는 이련의 자바 오브젝트를 위한 포괄적인 Wrapper를 말한다. 메시지는 여러개의 헤더로 구성된다.

- Enterprise Integration Patterns은 엔터프라이즈 환경에서 사용하고 있는 다양한 분야(ex. 결제,메일, 각 부서별 서비스) 의 애플리케이션을 통합, 즉 유기적으로 연결해서 효율적으로 적절하게 통합하는 방법을 여러 패턴을 통해 제시했다.
- 하나의 동작을 하는 서비스의 각각의 기능들(프로젝트)이 내/외부 모듈과 접촉하는 부분을 쉽게 구성할 수 있도록 하는 기능들을 제공한다.

### Spring Integration을 구성하는 컴포넌트 종류
- 채널: 한 요소로부터 다른 요소로 메시지를 전달
- 필터: 조건에 맞는 메시지가 플로우를 통과하게 해줌
- 변환기: 메시지 값을 변경하거나 메시지 페이로드의 타입을 다른 타입으로 변환
- 라우터: 여러 채널 중 하나로 메시지를 전달하며 대개 메시지 헤더를 기반으로 함
- 분배기: 들어오는 메시지를 두 개 이상의 메시지로 분할하며, 분할된 각 메시지는 다른 채널로 전송
- 집적기: 분배기와 상반된 것으로 별개의 채널로부터 전달되는 다수의 메시지를 하나의메시지로 결합함
- 서비스 액티베이터: 메시지를 처리하도록 자바 메서드에 메시지를 넘겨준 후 메서드의 반환값을 출력 채널로 전송
- 채널 어댑터: 외부 시스템에 채널을 연결함. 외부 시스템으로부터 입력을 받거나 쓸 수 있음
- 게이트웨이: 인터페이스를 통해 통합플로우로 데이터를 전달
## 3가지의 메인 Component
- Spring Integration은 'pipe and filters' 모델을 구현하기 위해 3가지 핵심 개념으로 구성되어 있다.
####  1. Message 
	header, payload로 구성되어 있는 내용을 포함하고 있는 generic wrapper. 
	컴포넌트 간에 이동되는 실제 데이터이다.

![[Pasted image 20240605160336.png]]

#### 2. Message Channel
	pipes-and-filters 모델의 pipe에 해당.
	컴포넌트간의 메세지 중간 통로 역할을 함으로써 컴포넌트간 디컬플링을 유지 할 수 있도록 하며 interception, monitering 포인트가 될 수 있다.

	다른 주요 기능 중 하나는, 메세지 버퍼 역할을 할 수 있는 Queue로써 동작할 수 있다.
	FIFO방식으로 컨슈머가 가져갈 때 까지 큐에 저장된다.
ex)
@Bean
public MessageChannel sampleChannel() {
	return new DirectChannel();
}
-> DirectChannel은 Point to Point로, 하나의 MessageHeader에게 Message를 전달한다.
이외에도 다양한 체널종류가 있다.


#### 3. Message Endpoint
	pipes-and-filters 모델의 filters에 해당.
	Spring integration상에서 채널을 통해서 메세지를 받고, 소비하는 주체이며 하나의 클래스이다.
	여기서 말하는 EndPoint란 Spring integration이 구성할 파이프라인의 끝단이 아닌, 파이프라인 중간에서 메세지를 변경하거나 필터링 하거나, 다른 채널로 라우팅하는 요소이다.
``` java
ex)
@MessageEndpoint
public class serverEndpoint {
	...
}
```

- Spring integration에서 일련의 작업들을 정의한 플로우를 integration flow 라고 하는데, 이 플로우가 Message Endpoint로 구성되어있다.

- 엔드포인트는 작업 타입에 따라 크게 그 종류를 나눌 수 있는데 Transformer(변형), Filter(필터링), Router(메세지를 특정 채널로 전송), Splitter(메세지를 분리하여 여러 채널로 전송), Aggregator(splitter의 반대), Service Activator(메세지로 특정 작업을 수행할 수 있는 핸들러를 붙일 수 있는 엔드포인트), Channel Adapter(외부 시스템과 입출력이 가능)가 있다.
## Filter 란?
- 통합 파이프라인 중간에 위치하며, 조건을 기반으로 플로우의 전 단계로부터 다음단계로의 메세지 전달에 조건을 달 수 있다.
ex)
``` java
@Filter( inputChannel="numberChannel", outputchannel="evenNumberChannel" )
public boolean evenNumberFilter( Integer number ) {
	return number % 2 == 0;      //숫자를 받아 짝수만 전달
}
```
## ServiceActivator 란?
- 입력체널로 부터 메세지를 수신하고, 이 메세지를 MessageHandler 인터페이스를 통해 구현한 클래스에 전달(서비스호출)한다.
- 서비스를 메시징 시스템에 연결하기 위한 앤드포인트이다.
- 입력 채널이 설정되어 있어야 하고, 서비스가 값을 리턴하도록 구현했다면 출력 채널도 설정해야한다.
- **만약 출력체널을 설정하지 않았을 때 메세지에 "return address"가 있다면 이 헤더에 지정한 체널로 응답을 전송한다.
- MessageChennel메서드를 입력해서 파이프라인을 구축한다,
``` java
ex)
@ServiceActivator(inputChannel = "sampleChannel") { }
```

---

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

# Spring MVC의 Service와 ServiceImpl 구조

- 해당 구조가 갖는 장점이 무엇인가?
먼저 Serviced에 인터페이스를 구현하여 세부 구현체를 숨기고 인터페이스를 바라보게 함으로써 클래스간의 의존관계를 줄이는것 이다.
좀 더 쉽게 정리하면,
	하나의 인터페이스를 구현하는 여러개의 구현체가 있고, 기능에 따라 적절한 구현체가 드어감으로써 다형성을 주기위함이다.

- 하지만, 인터페이스 하나에 구현체 한개만 사용하는경우는 어떠한가?
이렇게 된다면, 의존관계를 줄여주는 효과도,  다형성을 주는 효과도 없게된다.
	하지만 보통의 경우 한개의 기능을하는 인터페이스를 여러기능의 구현체로 나누는 일은 쉽게 일어나지 않는다.

``` java
public interface CardPaymentService {
    void pay();
}

public class ShinhanCardPaymentService implements CardPaymentService{
    private ShinhanCard shinhanCard;

    @Override
    public void pay() {
        shinhanCard.pay(); //신한 카드 결제 API 호출
        // 결제를 위한 비즈니스 로직 실행....
    }
}
```
### 하나의 인터페이스의 하나의 구현체
- 위와 같은 경우, 하나의 인터페이스에 하나의 구현체를 갖지만, 향후 추가적으로 구현체가 더 생길여지가 있으니, 인터페이스를 두는것이 바람직 하다고 할 수 있다.

그렇다면 향후 구현체가 추가될 계획이 없는 기능들 까지 인터페이스를 만들어주어야 하는가?
- 그렇지 않다, 예를들어 간단하게 아이디를 기반으로한 조회기능 등은 인터페이스를 구현하지 않고 바로 서비스 객체를 생성하는것이 옳다.
``` java
public interface ChangePasswordService {
    public void change(MemberId id, PasswordDto.ChangeRequest dto);
}

public class ByAuthChangePasswordService implements ChangePasswordService {
    private MemberFindService memberFindService;

    @Override
    public void change(MemberId id, PasswordDto.ChangeRequest dto) {
        if (dto.getAuthCode().equals("인증 코드가 적합한지 로직 추가...")) {
            final Member member = memberFindService.findById(id);
            final String newPassword = dto.getNewPassword().getValue();
            member.changePassword(newPassword);
            // 필요로직...
        }
    }
}

public class ByPasswordChangePasswordService implements ChangePasswordService {
    private MemberFindService memberFindService;

    @Override
    public void change(MemberId id, PasswordDto.ChangeRequest dto) {
        if (dto.getPassword().equals("비밀번호가 일치하는지 판단 로직...")) {
            final Member member = memberFindService.findById(id);
            final String newPassword = dto.getNewPassword().getValue();
            member.changePassword(newPassword);
        }
    }
}
```
이렇게 비밀번호를 변경하는 기능같은 경우에는 2가지 이상의 경우가 있기때문에 인터페이스로 구현하는것이 옳아보인다.

##### 결론 : 하나의 클래스에 너무 많은 역할을 부여하지 말자. (책임을 몰리지 말자)

---

# DTO ( Request와 Respons)
## Request ( Json to DTO ) 
	- Controller로 부터 값을 받는 객체로 사용.
## Responsse ( Object to Json)
 	- 엔티티로부터 타입 변환을 하여 Controller로 넘겨주는 객체


---

# 페이지 펙토리 객체 생성시, 리스트 넘길때 Page리스트 객체일 필요없다.
### PageFactory.paginate(==dList==, ((int) cnt), hfcfgLoanRequestDto)

``` java
Page<Dto>  -> List<Dto>
```

---

# 쿼리펙토리 빈으로 주입받은 것 바로 사용하기 ( 중복으로 생성할 필요 없음)
![[Pasted image 20240723134224.png]]

### 기존코드 : 
![[Pasted image 20240723134333.png]]

---

# Netty란??

## netty란 java기반의 네트워크 프레임워크이다.

### webClient도 Netty안에 포함되어 있다.

## 장점
- 비동기 적으로 입출력처리를 관리하기 때문에 전송 작업이 성공했는지 실패했는지 알수 있다.
- 비동기식 작업에서 높은 성능을 유지할수 있다.

---

# Equals() 와 HashCode() 재정의
``` java
@Test
@DisplayName("같은 객체를 equals 비교")
void equals() {
    //given
    Menu friedChicken = new Menu("후라이드치킨", 16_000);
    Menu friedChicken2 = new Menu("후라이드치킨", 16_000);
    //when & then
    assertThat(friedChicken).isEqualTo(friedChicken2);
}
```

헤당 코드와 같이 구현한다면, false를 출력한다.
이유는 객체의 equals메서드는 주소값이 서로 다른 객체는 다른객체로 판단하기 때문이다.
#### String은 equals 잘쓰잖아? => String은 String pool에서 중복생성을 막고, 같은 값을 생성한다면 재사용하기 때문~

## 재정의 해서, 객체가 아닌 내부 값을 비교하는 equals를 만들었다.
이때, 왜 HashCode도 재정의 해야하나?

### HashCode의 규약에는 다음과 같은 사항이 존재한다.

#### * equals(object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode값은 항상 같아야한다

해당 규약으로 인하여, 서로다른 객체의 해쉬값을 통일시켜주어야 한다.

