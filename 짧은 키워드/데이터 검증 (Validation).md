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