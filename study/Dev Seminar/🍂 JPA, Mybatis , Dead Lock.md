# 🍂 JPA, Mybatis , Dead Lock

#공부 #JPA #Java #Mybatis #DeadLock

---

*...Stack trace 중 일부 발췌*
```
HikariPool Dead lock

Caused by: org.apache.ibatis.exceptions.PersistenceException:
###Error querying database. Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
###The error occurred while executing a query
###Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
Caused by: java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30008ms.
at com.zaxxer.hikari.pool.HikariPool.createTimeoutException(HikariPool.java:696)
at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:197)
at com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:162)
```

``` java
// 위 조건 발생 코드 예시
public void deadLockMethod(){
    mybatisSelectMethod();    
    
    jpaSelectMethod();
    
    mybatisSelectMethod();
}
```

---

데드락 문제는 **데이터베이스 커넥션 고갈**과 **트랜잭션 리소스 경합**이 복합적으로 작용하여 발생합니다.

# 1. 데드락이 발생하는 주요 조건:

1. **공유 커넥션 풀(HikariCP)**:
    
    - JPA와 MyBatis 모두 HikariCP를 통해 동일한 데이터베이스 커넥션 풀을 공유합니다.
    - 한쪽에서 커넥션을 점유하고 반환하지 않으면, 다른 쪽에서 커넥션 요청 시 대기 상태가 길어지고, 결국 시간 초과(timeout)로 이어집니다.
2. **트랜잭션 관리의 복잡성**:
    
    - JPA와 MyBatis가 서로 다른 방식으로 커넥션을 관리하면서 동일 트랜잭션 내에서 서로 경합을 벌일 가능성이 큽니다.
    - 트랜잭션의 범위가 너무 크거나, 트랜잭션 경계에서 커넥션이 제대로 정리되지 않으면 커넥션 고갈 상태가 발생합니다.
3. **Nested Query 및 커넥션 잠금**:
    
    - `deadLockMethod()` 코드처럼 동일 메서드 내에서 JPA와 MyBatis를 교차 호출하면, 커넥션이 중복 점유(lock)될 가능성이 있습니다.
    - 예를 들어, MyBatis가 첫 번째 호출에서 커넥션을 점유한 상태에서 JPA가 새로운 커넥션을 요청하면, 풀에서 여유가 없을 경우 대기 상태가 발생합니다.

# 2. 데드락 발생 원인:

1. **JPA와 MyBatis의 동시 사용**
    
    - JPA는 ==엔터티 매니저(EntityManager)==를 통해 트랜잭션을 관리하고, MyBatis는 ==SQL 세션(SqlSession)==을 통해 관리합니다.
    - 이 둘은 서로 독립적으로 작동하기 때문에 동일한 트랜잭션 내에서 하나의 커넥션을 공유하지 못하고, 각자 별도의 커넥션을 요청할 수 있습니다.
2. **커넥션 반환 누락**
    
    - MyBatis 또는 JPA 메서드 호출 후 커넥션이 적절히 반환되지 않으면(pool로 반환되지 않음), 커넥션 풀이 고갈될 가능성이 높아집니다.
3. **Connection Pool 고갈**
    
    - `HikariCP`의 기본 `maxPoolSize`(기본값: 10)가 초과되면 대기 상태가 발생하며, 대기 시간(`connectionTimeout`)이 지나면 `SQLTransientConnectionException`이 발생합니다.



# 해결 방안 및 설계 지침

### 1. HikariCP 설정 조정

- **최대 커넥션 수 증가**: HikariCP의 `maximumPoolSize`를 늘려 충분한 커넥션을 사용할 수 있도록 설정합니다.
    
    properties
    
    `spring.datasource.hikari.maximum-pool-size=30 spring.datasource.hikari.connection-timeout=30000`
    
- **최소 유휴 커넥션 유지**:
    
    properties
    
    `spring.datasource.hikari.minimum-idle=10`
    

### 2. 트랜잭션 경계 명확히 하기

- JPA와 MyBatis 호출이 동일한 트랜잭션 내에서 실행되도록 **스프링 트랜잭션 관리**를 통합합니다.
- `@Transactional` 어노테이션을 적용하여 트랜잭션 범위를 명시적으로 정의합니다.
    
``` java
@Transactional
public void deadLockMethod() { 
	mybatisSelectMethod();
	jpaSelectMethod();
	mybatisSelectMethod();
}
```
### 3. 동일 커넥션 공유

- JPA와 MyBatis가 동일한 트랜잭션 안에서 **하나의 커넥션을 공유**하도록 설정합니다.
- Spring의 `DataSourceTransactionManager`를 사용하여 데이터 소스 기반으로 트랜잭션을 관리합니다.
    
``` java
    @Bean public PlatformTransactionManager transactionManager(DataSource dataSource) {
         return new DataSourceTransactionManager(dataSource);
    }
    
```

### 4. Lazy Loading 사용 최소화

- JPA에서 `Lazy Loading`이 과도하게 사용될 경우, 예상치 못한 시점에서 데이터베이스 호출이 발생하며 추가적인 커넥션 요청을 유발할 수 있습니다.
- 가능하면 즉시 로딩(`Eager Fetch`)을 사용하거나, 필요한 데이터를 명시적으로 로딩합니다.
    
``` java
@EntityGraph(attributePaths = {"childEntities"}) List<ParentEntity> findWithChildren();
```
    

### 5. 커넥션 반환 명시

- MyBatis 사용 시 `SqlSession`을 명시적으로 닫아 커넥션이 즉시 반환되도록 합니다.
    
    `try (SqlSession session = sqlSessionFactory.openSession()) {     // MyBatis 작업 수행 }`

``` java
@Service
public class DeadlockService {

    private final SqlSessionFactory sqlSessionFactory;
    private final JpaRepository jpaRepository;

    public DeadlockService(SqlSessionFactory sqlSessionFactory, JpaRepository jpaRepository) {
        this.sqlSessionFactory = sqlSessionFactory;
        this.jpaRepository = jpaRepository;
    }

    @Transactional
    public void resolveDeadlockMethod() {
        // MyBatis 호출
        try (SqlSession session = sqlSessionFactory.openSession()) {
            session.selectList("namespace.mybatisSelect");
        }

        // JPA 호출
        jpaRepository.findAll();

        // 다시 MyBatis 호출
        try (SqlSession session = sqlSessionFactory.openSession()) {
            session.selectList("namespace.mybatisSelect");
        }
    }
}
```


--- 

# 헬로에서의 데드락 원인

### 하나의 동작(트렌젝션)에서 커넥션을 하나 점유한 상태에서,
### 추가적으로 요구하고, 추가적으로 요구된 커넥션풀을 다른 트렌젝션에서 점유하고있을때.


# JPA의 영속성 컨텍스트가 커넥션을 놓는 순간?
## Entity Manger -> 영속성 컨텍스트
#### OSIV(Open Session In View)


---

# 정리글
[[ 트러블슈팅 ] JPA, Mybatis 동시 사용시 발생할 수 있는 HikariCP Dead lock 해결 여정 ( feat.OSIV )](https://velog.io/@12onetwo12/%EC%9E%A5%EC%95%A0%ED%9A%8C%EA%B3%A0-DBCP-Connection-Leak-%ED%95%B4%EA%B2%B0-%EC%97%AC%EC%A0%95-feat.HikariCP-Dead-lock-QueryDSL)

