# 🍎 Redis로 Key관리하기

#공부 #Redis #Cash #memory #다중서버 #NoSql

---

# *Redis*란 무엇일까?
<br/>

## Redis(원격 디렉터리 서버 : **Re**mote **Di**ctionary **S**ever)
- 주로 애플리케이션 캐시 또는 빠른 응답 데이터베이스로 사용되는 오픈소스,
인매모리, NoSql <키, 벨류> 저장소이다.

또한, **redis**는 보조기억장치(HDD / SSD)가 아닌 <u>메모리에 데이터를 저장</u>하여
탁월한 속도, 안정성, 성능을 제공할 수 있다.

## Redis의 적용 목적

 애플이케이션이 외부 데이터 소스에 의존하는 경우 <u>트레픽이 증가하거나, 애플리케이션이 확장될때 소스의 지연 시간과 처리량으로 인하여 병목현상이 발생할 수 있다</u>.
 이때 Redis를 적용하면, 데이터를 메모리에 저장하여 읽거나 쓸때 **지연 시간**을 최소화 할 수 있다.

---

# *Redis*의 기능

- redis는 앱 성능 향상을 위해 특별히 설계되어, 기존 NoSQL 데이터 저장소와 차별화 되는 기능이 있다.

### 1. Redis 캐시 세션

 - MongoDB, PostgreSql 과 같은 NoSQL DB와 달리, 메모리를 저장소로 사용하여
 읽기 쓰기 성능이 월등히 높다. 또한 고가용성과 확장성을 보장하는데 도움이된다.

>[!tip] 고가용성 이란?
> 가용성 : 서버 또는 네트워크 등의 정보 시스템이 정상적으로 사용 가능한 정도를 의미.
>   == `정상적인 사용시간` / `전체 사용시간`  = `시스템 가동률`(가용성)
>   
>   여기서, 고가용성이란 가용성이 99%, 99.9% 등과 같이 높은 가용성을 지닌 시스템을 의미한다.

### 2. Redis 대기열

- redis는 웹 클라이언트가 평소보다 처리하는데 오래 걸릴 수 있는 작업을 대기열에 넣을 수 있다.
 요청/응답 주기의 백그라운드에서 실행되는 자동화된 프로세스를 쉽게 구현할 수 있는 것이다.

### 3. Redis 데이터 형식

- redis는 기술적으로는 키/벨류 저장소 이지만, 여러 데이터 유형과 구조를 지원하는 데이터 구조 서버이다.

#### 지원 데이터 ex)
1. 고유하고 정렬되지 않은 문자열
2. 바이너리 세이프 데이터
3. 하이퍼로그
4. 비트 배열
5. 해시
6. 목록

---

# - *RSA private key* 암호화 방식 적용하기
- #### RSA란 무엇인가? ▶ [[🚨 RSA 암호화 방식의 이해와 적용 (feat.취약성점검)]]

---

# - 캐쉬 전략으로 **Redis**를 선택한 이유?
- #### 인메모리 캐시와의 비교 ▶ [[🤲분산 환경에서의 Cache 선택하기]]

---

# **Redis** 연결 및 구현 With. Spring
<br/>


# 1. 환경 설정

**build.gradle에 의존성 추가**  

```java
implementation 'org.springframework.boot:spring-boot-starter-data-redis'  
```

**yml redis 속성 추가**  
```yml
cache:  
type: redis  
redis:  
cache-null-values: true

redis:  
host: `레디스 호스트`
port: `레디스 포트`
```

---

## 2. RedisConfiguration
- @Configuration으로 redis사용에 필요한 셋팅을 Bean으로 등록할 클래스.
```java
@Configuration  
public class RedisConfiguration {  
  
    @Value("${spring.redis.host}")  
    private String host;  
  
    @Value("${spring.redis.port}")  
    private int port;
}
```
<br/>

---

## **RedisConnectionFactory**

Redis 서버와 연결을 생성 및 관리해주는 인터페이스
```java
@Bean  
public RedisConnectionFactory redisConnectionFactory() {  
    return new LettuceConnectionFactory(host, port);  
}
```
- 어플리케이션 서버와 Redis 서버 간의 데이터 송수신을 하는 클라이언트
- 대표적으로 *Lettuce*와 *Jedis*, *Redisson* 이 있다.

### Lettuce
- 비동기 및 논블로킹 I/O를 기반으로 하여 고부하, 다중 스레드 환경에 적합

### Jedis
- 블로킹 I/O(동기) 방식을 사용.
- 고부하나 비동기 처리가 중요한 환경에서는 효울이 떨어진다.

### redisson
- 단순히 Redis 연결을 관리하는 것을 넘어 <u>분산 락, 분산 컬렉션, 분산 캐시</u> 등 고급 기능을 제공.
- 직접 <u>RedisConnectionFactory</u>로 사용하기보다는 *RedissonClient를 빈으로 등록*하고 이를 통해 분산 락이나 캐시 매니저를 구성.
- #### redisson 사용 예
   ▶ [[🔐 상품 투자하기 서비스 Lock기법 개선안]]
<br/>

따라서 해당 코드에는 비동기 성능이 높은 좋은 **Lettuce** 선택.

---

## **RedisCacheDefaultConfiguration**

Redis에 저장될 캐시의 <u>기본</u> 직렬화 및 만료 시간(TTL) 등의 설정을 담당.

```java
private RedisCacheConfiguration redisCacheDefaultConfiguration() {
    return RedisCacheConfiguration
            .defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                    .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer());
}
```

- *serializeKeysWith* : `StringRedisSerializer`를 사용하여 키를 문자열로 직렬화합니다.
- *serializeValuesWith* : `GenericJackson2JsonRedisSerializer`와 `ObjectMapper`를 사용해 JSON 형식으로 직렬화
> `GenericJackson2JsonRedisSerializer` : 직렬화 방식 중 하나로, JSON형식을 지원.

---

## **redisCacheConfigurationMap**

여러 캐시 이름에 대해 각기 다른 TTL(Time To Live)을 동적으로 설정.

```java
private Map<String, RedisCacheConfiguration> redisCacheConfigurationMap() {  
    Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();  
    for (Map.Entry<String, Long> cacheNameAndTimeout : cacheProperties.getTtl().entrySet()) {  
        cacheConfigurations  
                .put(cacheNameAndTimeout.getKey(), redisCacheDefaultConfiguration().entryTtl(  
                        Duration.ofSeconds(cacheNameAndTimeout.getValue())));  
    }  
    return cacheConfigurations;  
}
```
<br/>

//cacheProperties.yml
```yml
cache:  
  ttl:  
    CacheName: 10 #만료 시간
```
- 외부 설정(`CacheProperties`)에서 캐시별 TTL 정보를 읽어와 각 캐시의 만료 시간을 지정
이를 통해 특정 캐시만 별도의 만료 정책 등을 적용할 수 있다.
- *entryTtl* : 기본 만료시간 설정

---

## **RedisCacheManager**  

Spring의 캐시 추상화에서 Redis를 캐시 저장소로 사용하기 위한 캐시 매니저를 생성
위에서 설정한 `redisCacheDefaultConfiguration`과 `cacheConfigurations`(커스텀) 이 삽입된다.

```java
@Bean
public CacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
    return RedisCacheManager.RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory)
            .cacheDefaults(redisCacheDefaultConfiguration())
            .withInitialCacheConfigurations(redisCacheConfigurationMap())
            .build();
}
```

- `withInitialCacheConfigurations` : *RedisCacheManager*를 생성할 때 <u>미리 정의된 특정 캐시 이름</u>에 대해 개별적인 설정을 적용할 수 있도록 해주는 메서드입니다.


---

# 3. Service Layer
▶ [[🚨 RSA 암호화 방식의 이해와 적용 (feat.취약성점검)]] 에서 이어진다.

## *Bean 주입*
```java
@Service  
public class TestServiceImpl {

	private final CacheManager cacheManager;  
	private final RedisTemplate<String, Object> redisTemplate;

	public TestServiceImpl(CacheManager cacheManager, RedisTemplate<String, Object> redisTemplate) {  
	    this.cacheManager = cacheManager;  
	    this.redisTemplate = redisTemplate;  
	}
}
```
(생성자 주입)
- 위(*RedisConfiguration*)에서 생성한 CacheManager 및 RedisTemplate의 빈을 주입한다.

>[!info] 주의
> 
> 만약, Bean으로 생성된 CacheManager객체나, RedisTemplate객체가 여러개라면,
> @Qualifier 어노테이션으로 Bean이름을 명시해야한다.
> >
> 	`ex) @Qualifier("CustomCacheManager") CacheManager cacheManager ...`

---

##  *Cache 삽입 / 꺼내기 / 삭제 *

```java
	Cache privateKeyCache = cacheManager.getCache("CacheName");
	
	public void putCache() {
		
		if (privateKeyCache != null) {  
		    privateKeyCache.put(keyId, 벨류);  
		} else {  
		    // 캐시가 없으면 예외 처리 또는 로깅  
		    throw new IllegalStateException("privateKeyCache 가 유요하지 않습니다.");  
		}
	}

	public void getCache() {
		
		if (privateKeyCache == null) {  
			throw new IllegalStateException("rsaPrivateKeyCache 가 유요하지 않습니다.");  
		}  
		String privateKeyValue = privateKeyCache.get(keyId, String.class);  
		// 1회용 사용을 위해 조회 후 캐시에서 제거할 수 있다.
		rsaPrivateKeyCache.evict(keyId); // 1회용 사용: 캐시에서 제거
		
	}
```

- getCache.(*CacheName*)으로 캐쉬를 객체를 가져온다.
- `put(keyId, 벨류);` / `get(keyId, String.class);` 로 삽입 / 가져오기가 가능하다.
- `.evict(keyId)`로 삭제 ( 1회성 사용이 가능하다. )

>[!info] 1회성으로 사용하는 이유
>
> 나의 경우에 RSA키를 매번 발급 받기 때문에 값을 꺼냄과 동시에 해당 키벨류를 삭제한다.
> Exception이 터지더라도, cacheProperties 에 설정한 TTL이 초과되면 삭제된다.

---

#### Redis서버를 사용한 Key 관리로, 멀티 서버 환경에서 정합성과 안정성을 챙길 수 있었다.

## + TTL 체크
![[Pasted image 20250328120415.png]]

# + TTL 설정

```c
application.yml / properties
      ↓
CacheProperties (ttl map 관리)
      ↓
redisCacheConfigurationMap() → 캐시별 TTL 매핑
      ↓
redisCacheDefaultConfiguration() → 기본 설정 (ex: serializer, 기본 TTL)
      ↓
redisCacheManager() → 최종 CacheManager 생성
```
다음과 같이 TTL 시간 및 [[Redis 만료 정책]]을 맵핑 할 수 있다.
