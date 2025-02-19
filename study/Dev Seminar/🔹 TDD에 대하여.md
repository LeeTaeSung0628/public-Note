# 🔹 TDD에 대하여

#공부 #Java #TDD #Test

---

# TDD가 필요한 이유

1. **코드 품질 향상**
    
    - TDD는 먼저 테스트를 작성한 후 코드를 구현하므로, 요구사항이 코드로 명확히 반영됩니다.
    - 코드의 결함이 초기에 발견되어 수정 비용이 감소합니다.
2. **유지보수성 강화**
    
    - 잘 작성된 테스트는 코드가 리팩토링될 때 문제를 방지합니다.
    - 신규 기능 추가 시, 기존 코드와의 충돌을 빠르게 감지할 수 있습니다.
3. **명확한 설계 유도**
    
    - TDD는 코드를 작고 독립적인 단위로 나누어 설계하도록 장려합니다.
    - 자연스럽게 SRP(Single Responsibility Principle) 등 객체지향 원칙을 따르게 됩니다.
4. **개발자 간 협업 강화**
    
    - 테스트는 개발자들 간의 명세서 역할을 하며, 코드의 동작을 명확히 설명합니다.
    - 코드리뷰 시 테스트를 통해 의도한 동작을 검증할 수 있습니다.

---

## TDD 적용 범위?

1. **적용이 적합한 상황**
    
    - **핵심 비즈니스 로직**: 제품의 주요 기능이나 서비스 로직은 테스트가 필수적입니다.
    - **복잡한 계산이나 알고리즘**: 오작동이 치명적인 로직에는 TDD가 효과적입니다.
    - **API 개발**: REST API나 GraphQL과 같은 인터페이스는 TDD로 사전 정의하여 일관성을 보장합니다.
    - **CI/CD 파이프라인**: 자동화된 테스트를 활용하여 배포 전에 코드를 검증합니다.
2. **적용이 어려운 상황**
    
    - **UI/UX**: 사용자 인터페이스는 변동이 잦아 TDD를 적용하기 어렵습니다. 대신 e2e 테스트를 고려할 수 있습니다.
    - **초기 프로토타입 개발**: 초기에는 빠른 구현이 우선이므로 TDD를 생략할 수 있습니다.
    - **시간/리소스 부족**: 모든 코드를 테스트하기 어려운 경우, 핵심 영역에 우선순위를 두어야 합니다.
3. **테스트의 범위 조정**
    
    - **유닛 테스트**: 가장 작은 단위의 코드 동작을 테스트합니다.
    - **통합 테스트**: 여러 모듈이 함께 작동하는지 확인합니다.
    - **엔드투엔드 테스트(e2e)**: 사용자가 시스템 전체를 사용할 때의 흐름을 테스트합니다.

---

## 실무 팁: TDD를 효과적으로 적용하려면?

1. **우선순위를 정하라**
    
    - 모든 코드를 테스트하기보다는 핵심적인 기능에 집중합니다.
    - 낮은 리스크 영역은 이후에 커버리지를 추가할 수 있습니다.
2. **자동화와 병행하라**
    
    - CI/CD 환경에서 테스트가 자동으로 실행되도록 설정하여, 코드 품질을 지속적으로 유지합니다.
3. **테스트를 문서로 활용하라**
    
    - 테스트는 단순히 검증 도구가 아니라, 코드의 의도를 설명하는 문서 역할을 합니다.
    - 
# 필요하다면 실무에서의 범위는 어디까지일까?
## 1. **핵심 비즈니스 로직**

- **사례**: 금융 계산, 결제 시스템, 인증 시스템.
- **이유**: 로직 오류가 사용자 신뢰에 큰 영향을 미칠 수 있으며, 수정 비용이 크기 때문.
- **장점**:
    - 요구사항 변경 시 빠르게 대응 가능.
    - 결함 발생 확률 감소.
- **TDD 도입 판단 기준**:
    - 로직 복잡도: 복잡할수록 도입 우선순위가 높음.
    - 장애 비용: 장애 발생 시 손실이 크면 반드시 도입.

---

## 2. **대규모 협업 프로젝트**

- **사례**: 여러 팀이 API를 개발하고 사용하는 상황.
- **이유**: 인터페이스 사양이 변경되면 다른 팀에 영향을 줄 수 있음.
- **장점**:
    - 명확한 명세서 제공으로 의사소통 오류 감소.
    - 코드 변경 시, 관련 기능이 깨지지 않도록 보호.
- **TDD 도입 판단 기준**:
    - 팀 간 의존도가 높을수록 도입 가치 증가.
    - 계약 기반 API 설계(contract-first approach)에 적합.

---

## 3. **복잡한 알고리즘**

- **사례**: 추천 시스템, 머신러닝 모델 전처리 로직.
- **이유**: 알고리즘의 정확도가 비즈니스 성과와 직결.
- **장점**:
    - 다양한 입력 데이터에 대한 처리 검증 가능.
    - 결과의 일관성 유지.
- **TDD 도입 판단 기준**:
    - 입력/출력 데이터의 조합이 복잡할수록 필요성 증가.
    - 정형화된 테스트 케이스 작성이 용이한 경우 적합.

---

## 4. **프로토타입 개발**

- **사례**: 스타트업 초기 제품.
- **이유**: 빠른 피드백과 시장 검증이 우선.
- **단점**:
    - 초기에는 개발 속도가 저하될 수 있음.
    - 불필요한 테스트 작성 가능성.
- **TDD 도입 판단 기준**:
    - 제품 성공 가능성이 높은 핵심 기능에만 제한적으로 적용.
    - 테스트는 리팩토링 이후 작성하는 방식을 병행.

---

## 5. **긴 수명 주기를 가진 프로젝트**

- **사례**: 장기 유지보수가 필요한 레거시 시스템.
- **이유**: 시간이 지나도 코드 품질을 유지해야 함.
- **장점**:
    - 기존 코드를 안전하게 리팩토링 가능.
    - 신규 개발자 온보딩 시 유용한 학습 자료 제공.
- **TDD 도입 판단 기준**:
    - 유지보수 비용이 크거나, 새로운 요구사항이 자주 추가되는 프로젝트.

# 테스트 툴
## 1. **JUnit 5 (Jupiter)**

JUnit 5는 단위 테스트(Unit Test)를 작성하기 위한 핵심 도구입니다. Spring Boot는 기본적으로 JUnit 5를 지원하며, 다양한 애너테이션과 Assertions API를 통해 테스트를 쉽게 작성할 수 있습니다.

### 주요 애너테이션

1. `@Test`
    - 테스트 메서드를 정의.
2. `@BeforeEach` / `@AfterEach`
    - 각각 테스트 메서드 실행 전후에 실행될 로직 정의.
3. `@BeforeAll` / `@AfterAll`
    - 각각 테스트 클래스 실행 전후에 한 번만 실행.
4. `@Disabled`
    - 테스트 메서드나 클래스를 비활성화.
5. `@Tag`
    - 테스트 그룹화(tagging) 기능 제공.

### Assertions API

- **`assertEquals(expected, actual)`**: 두 값이 동일한지 비교.
- **`assertTrue(condition)`**: 조건이 참인지 확인.
- **`assertThrows(exception, executable)`**: 특정 예외가 발생하는지 확인.

### 예제: 간단한 JUnit 테스트
``` java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @Test
    void additionShouldBeCorrect() {
        Calculator calculator = new Calculator();
        int result = calculator.add(2, 3);
        assertEquals(5, result, "2 + 3은 5여야 합니다.");
    }

    @Test
    void divisionShouldThrowExceptionForZero() {
        Calculator calculator = new Calculator();
        assertThrows(ArithmeticException.class, () -> calculator.divide(5, 0));
    }
}
```
### 장점

- **빠른 실행**: 단위 테스트는 다른 계층과 독립적이므로 빠르게 실행 가능.
- **간단한 설정**: 별도의 컨텍스트 로딩 없이 메서드 단위로 검증 가능.

---

## 2. **Spring Test Framework**

Spring Test Framework는 Spring Boot 애플리케이션의 통합 테스트를 지원합니다. 스프링 컨텍스트를 로드하여 Spring Bean, 의존성 주입, 데이터베이스 연동 등을 테스트할 수 있습니다.

### 주요 애너테이션

1. **`@SpringBootTest`**
    
    - 전체 애플리케이션 컨텍스트를 로드.
    - 단위 테스트와 통합 테스트를 모두 지원.
    - ```
``` java
@SpringBootTest
class MyAppTests {
    @Test
    void contextLoads() {
    }
}
```

2. **`@WebMvcTest`**

- 컨트롤러 테스트 전용으로 사용.
- MVC 관련 빈만 로드하여 REST API 테스트에 최적화.
``` java
@WebMvcTest(controllers = MyController.class)
class MyControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnDefaultMessage() throws Exception {
        mockMvc.perform(get("/api/hello"))
               .andExpect(status().isOk())
               .andExpect(content().string("Hello, World!"));
    }
}

```

3. **`@DataJpaTest`**

- JPA와 관련된 빈만 로드하여 테스트 속도를 최적화.
- H2 Database와 함께 사용하여 데이터 계층 테스트.
``` java
@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldSaveAndFindUser() {
        User user = new User("John Doe", "john@example.com");
        userRepository.save(user);

        Optional<User> result = userRepository.findByEmail("john@example.com");
        assertTrue(result.isPresent());
        assertEquals("John Doe", result.get().getName());
    }
}

```

4.**`@MockBean`**

- 특정 빈을 모킹하여 테스트 시 실제 구현체 대신 사용할 수 있음.
``` java
@SpringBootTest
class MyServiceTest {
    @MockBean
    private MyRepository myRepository;

    @Autowired
    private MyService myService;

    @Test
    void shouldUseMockedRepository() {
        when(myRepository.findSomething()).thenReturn("Mocked Result");

        String result = myService.getSomething();
        assertEquals("Mocked Result", result);
    }
}

```

---

# 최근 자체 디도스(내부 로직 부하)에 대한 대책

## **1. JMeter로 JPA 쿼리 부하 테스트 준비**

### **필수 요구사항**

1. **JDBC 드라이버 준비**
    
    - 테스트할 데이터베이스의 JDBC 드라이버를 다운로드하여 JMeter의 `lib` 디렉토리에 복사.
        - 예: MySQL은 `mysql-connector-java`, PostgreSQL은 `postgresql.jar`.
2. **테스트 대상 애플리케이션 준비**
    
    - JPA가 사용하는 데이터베이스를 직접 테스트하려면 데이터베이스 접속 정보가 필요.
    - Spring Boot 애플리케이션을 통해 동적으로 데이터를 생성하고 쿼리를 실행.

---

## **2. JMeter에서 JDBC Request 설정**

### **1단계: JDBC Connection Configuration 추가**

1. JMeter의 **Thread Group**에 **JDBC Connection Configuration**을 추가.
2. 설정:
    - **Variable Name**: 커넥션을 참조할 이름 (예: `DBConnection`).
    - **Database URL**: JDBC URL (예: `jdbc:mysql://localhost:3306/testdb`).
    - **JDBC Driver class**: 드라이버 클래스 이름 (예: `com.mysql.cj.jdbc.Driver`).
    - **Username/Password**: 데이터베이스 접속 정보.

---

### **2단계: JDBC Request 추가**

1. Thread Group에 **JDBC Request**를 추가.
    
2. 설정:
    
    - **Variable Name**: 이전 단계에서 설정한 Variable Name 입력 (`DBConnection`).
    - **Query Type**:
        - `Select Statement`: 데이터 조회 쿼리.
        - `Update Statement`: 데이터 수정 쿼리.
    - **Query**: 실행할 SQL 쿼리.
        - 예: `SELECT * FROM users WHERE status = 'ACTIVE';`
3. 매개변수를 동적으로 설정하려면 **Prepared Statement**를 사용할 수 있음:
    
    - SQL: `SELECT * FROM users WHERE age > ?;`
    - Parameters: `30` (동적으로 전달할 값).

---

### **3단계: 동적 부하 생성**

1. **Thread Group** 설정:
    
    - **Number of Threads (Users)**: 동시 실행 사용자 수.
    - **Ramp-Up Period**: 사용자가 몇 초에 걸쳐 증가할지 설정.
    - **Loop Count**: 각 사용자가 실행할 요청 반복 횟수.
2. 부하를 동적으로 변경:
    
    - 스레드 그룹에서 **Scheduler**를 활성화하여 특정 시간 간격으로 부하를 증감 가능.

---

## **3. JMeter와 JPA 연동 전략**

### **JPA 애플리케이션 직접 테스트**

1. **HTTP 요청을 통해 JPA 메서드 호출**
    
    - REST API 엔드포인트를 호출하여 JPA 쿼리 실행.
    - JMeter의 HTTP Sampler로 테스트:
        - URL: `http://localhost:8080/api/users?status=ACTIVE`
        - Method: GET/POST.
2. **JPA 쿼리 매개변수 동적 설정**
    
    - REST API에 쿼리 매개변수 전달.
    - JMeter의 CSV Data Set Config를 사용하여 다양한 입력값 시뮬레이션:
        - CSV 파일: `age,status`
            
            `25,ACTIVE 30,INACTIVE 35,ACTIVE`
            
        - 테스트 설정:
            - Parameter: `${age}`와 `${status}`로 대체.

---

## **4. 결과 분석**

### **JPA 쿼리 성능 확인**

1. **JMeter View Results Tree**
    
    - 쿼리 실행 결과 및 응답 시간 확인.
2. **Database Monitoring Tools**
    
    - 데이터베이스의 실행 계획(EXPLAIN) 확인.
    - DB에서 쿼리 실행 시간, CPU 사용량, I/O 병목 파악.
3. **Spring Actuator Metrics**
    
    - `@Timed` 애너테이션 또는 Actuator 메트릭을 통해 JPA 메서드의 실행 시간 측정.

---

## **5. 부하 테스트 시 주의사항**

1. **데이터베이스 상태 초기화**
    
    - 테스트 전에 데이터베이스 상태를 초기화하여 결과의 일관성 유지.
2. **실제 환경과 동일한 설정**
    
    - 실제 운영 환경의 DB 크기, 커넥션 풀 크기, 네트워크 조건을 반영.
3. **적정 부하 설정**
    
    - TPS(Transactions Per Second)를 기준으로 동시 요청 수 결정