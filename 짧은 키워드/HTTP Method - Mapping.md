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