# 📘 SpringBoot & Docker + Reids 연동

#Tools #도커 #Docker #SPRING #Boot #Redis 

---
![[Pasted image 20250221114306.png]]

먼저 도커에 대한 간단한 이전 글!
# [[🐋 docker]]

aws의 설명에 의하면 Docker란 
1. 애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼이다. 
2. 소프트웨어를 컨테이너 라는 표준화된 유닛으로 패키징하여 관리한다.

#### 이번 시간에는 스프링 부트 프로젝트에 Docker를 연동하여 local 서버를 만들어보겠다.

## 1. Docker다운로드 받기

[도커 다운로드 링크](https://docs.docker.com/desktop/install/mac-install/)

각 기기에 맞는 docker 버전을 다운로드 받는다.
![[Pasted image 20250221114324.png]]

다운로드 받은 후 명령어를 입력하여 docker의 현재 버전을 볼 수 있다.

※ 터미널에 'docker' 를 입력하게 되면 다양한 명령어들을 확인할 수 있다.
![[Pasted image 20250221114327.png]]

제대로 다운로드 받았다면 docker에 기본적으로 만들어져있는 이미지들을 확인 할수 있다. 밑에 사진은 도커 데스트톱 이라는 앱이다.

![[Pasted image 20250221114334.png]]

터미널에서도 확인할 수 있다.

![[Pasted image 20250221114339.png]]

이제 **도커 파일** 을 이용하여 **도커 이미지**를 생성한 후, 이미지를 빌드하여 **컨테이너**를 생성해 줄것이다.

여기서 여러 용어들이 나오는데...

용어들에 대한 설명이다

![[Pasted image 20250221114345.png]]

## 2.  Spring 프로젝트 내에 DockerFile 생성하기

![[Pasted image 20250221114353.png]]

아주 기본적인 Spring 프로젝트이다.

Dockerfile을 생성하기에 앞서, .jar 파일을 생성해야한다.

.jar파일을 생성하는데에 gradle 이나 maven 을 사용하여 프로젝트를 빌드 할것이다.

여기서는 Maven 을 사용한다.

[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)


홈페이지에서 직접 다운받아도 되고, brew를 사용해 각자 다운받으면 된다.

이후에, 프로젝트의 루트 디렉토리에서 'mvn install' 명령어를 사용하여 프로젝트를 빌드 할 수 있다.

![[Pasted image 20250221114400.png]]

빌드를 마치면, 프로젝트 내에 **target**이 생성 됨을 볼 수 있다.

![[Pasted image 20250221114406.png]]

그 후, 루트 디렉토리에 'Dockerfile' 을 생성하여 준다.

![[Pasted image 20250221114411.png]]

![[Pasted image 20250221114423.png]]

Dockerfile 생성시 사용되는 명령어들은 다음과 같다.

![[Pasted image 20250221114428.png]]

나는 기본적인 옵션만 넣어서 test 하였다.

**FROM 태그에 나의 java(jdk)버전을 확인하여 baseimage를 지정하고**

( dockerRpository에서 다양한 이미지를 가져다 사용할 수 있으므로 찾아보세요)

**ADD 태그에 maven을 통해 생성된 .jar파일과 경로를 입력한다**

**이외에 태그는 자유롭게 작성해 줄 수 있다.**

## 3. dockerfile 을 실행하여 이미지를 생성한다.

프롬프트에 'docker build -t ${"도커파일 명"} ${"디렉토리"} 형식으로 사용할 수 있다.

![[Pasted image 20250221114434.png]]

![[Pasted image 20250221114439.png]]

해당 단계를 마치면 이미지가 생성된 것을 확인할 수 있다.

## 4. 마지막으로, 생성된 이미지을 RUN 명령어를 통해  컨테이너로 실행하기!!

프롬프트에 'docker run -p ${사용할 포트번호} ${이미지 명}' 을 사용한다.

본인은 8000번 포트를 직접 사용하여 8080포트인 도커 컨테이너에 접근할 것이다.

![[Pasted image 20250221114443.png]]

짜잔!   정상적으로 동작한다.

이제 설정한 포트로 도커 컨테이너에 접근 할 수 있다.
![[Pasted image 20250221114501.png]]
![[Pasted image 20250221114448.png]]

---

# + 같은 방법으로 Redis 연동하기!(실습)


## Step 1. Docker를 이용하여 Redis를 받아보자!

docker는 다운받아져 있는걸로 알고.. 

'docker pull redis:latest' 명령어를 사용하여 최신버전의 redis를 받아온다.

![[Pasted image 20250221114522.png]]

그 다음, 'docker network create redis-network --driver bridge' 명령어를 사용하여 네트워크를 생성해 준다.

**※ docker network : 컨테이너간의 통신 및 데이터 공유를 위한 가상 네트워크**

![[Pasted image 20250221114527.png]]

이제, 'sudo vim redis.conf' 명령어를 사용하여 redis.conf 파일을 수정하여 설정을 잡아준다.

![[Pasted image 20250221114531.png]]

```
docker run \
        -d \
        --name redis \
        -p 6379:6379 \
        --network redis-network \
        -v ~/${데이터를 저장할 파일 경로} /redis.conf:/etc/redis/redis.conf \
        -v redis_data:/data \
redis:latest redis-server /etc/redis/redis.conf
```

![[Pasted image 20250221114535.png]]

포트번호와, 방금 설정한 네트워크, 데이터 저장 경로를 잡아준다.

이렇게 설정을 마쳤으면 redis 이미지가 생성되었음을 확인할 수 있다.

![[Pasted image 20250221114539.png]]

최종적으로 생성된 이미지로 컨테이너를 만들어 준다.

![[Pasted image 20250221114543.png]]

```
docker run --name myredis \
        -p 6379:6379 \
        --network redis-network \
        -v /Users/lts/Desktop/docker/redis:/data \
        -d redis:latest redis-server \
        --appendonly yes
```

![[Pasted image 20250221114600.png]]

**컨테이너까지 생성완료!**

'docker exec -it myredis redis-cli --raw' 명령어를 사용하여 컨테이너 실행

![[Pasted image 20250221114605.png]]

프롬프트를 보면 정상적으로 접속됨을 확인할 수 있다.

## Step 2. Redis와 Spring Boot 연동하기

※ 해당 프로젝트는 Maven 을 사용한 프로젝트이다.

### 먼저 application.properties 에 redis 포트를 추가하여준다

```
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379
```

![[Pasted image 20250221114610.png]]

### 그 다음 config 파일을 작성하여준다.

![[Pasted image 20250221114616.png]]

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        final StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setKeySerializer(new StringRedisSerializer());
        stringRedisTemplate.setValueSerializer(new StringRedisSerializer());
        stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
        return stringRedisTemplate;
    }
}
```

### Model과 Repository 도 작성해준다.

![[Pasted image 20250221114621.png]]

```
import lombok.AllArgsConstructor;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.redis.core.RedisHash;
import org.springframework.data.redis.core.index.Indexed;

import java.time.LocalDateTime;

@Getter
@RedisHash(value = "resultHistory", timeToLive = 3600) // Redis Repository 사용을 위한
@AllArgsConstructor
@NoArgsConstructor
public class ResultHistory {

    @Id
    private String id;
    @Indexed // 필드 값으로 데이터를 찾을 수 있도록 설정 (findByAccessToken)
    private String ip;
    private String originalText;
    private String translatedText;
    @Indexed
    private LocalDateTime createDateTime;

    @Builder
    public ResultHistory(String ip, String originalText, String translatedText, LocalDateTime createDateTime) {
        this.ip = ip;
        this.originalText = originalText;
        this.translatedText = translatedText;
        this.createDateTime = createDateTime;
    }
}
```

---

![[Pasted image 20250221114628.png]]

```
import com.teamTS.funfun.model.ResultHistory;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface ResultRedisRepository extends JpaRepository<ResultHistory, String> {
    Optional<List<ResultHistory>> findByIpOrderByCreateDateTimeAsc(String ip);
}
```

### controller 에 간단한 테스트 코드를 작성하여 값을 확인해보자!

![[Pasted image 20250221114633.png]]

```
    @GetMapping("/hello")
    public String HelloWorld(Model model) {
    
        redisConnectionTest();
        
        List<TestModel> tm = testRepository.getTestData();
        model.addAttribute("data", tm.get(0).getTitle());
        return "home/homeView";
    }

    void redisConnectionTest() {
        final String key = "a";
        final String data = "1";

        final ValueOperations<String, String> valueOperations = redisTemplate.opsForValue();
        valueOperations.set(key, data);

        final String s = valueOperations.get(key);
    }

}
```

## **그런데...**

![[Pasted image 20250221114638.png]]

'Error starting ApplicationContext. To display the condition evaluation report re-run your application with 'debug' enabled.'

#### 대충 빈 생성이 중복으로 발생하여 생긴 오류인듯 하다..

원인은 기존 MapperRepository와 Redis용 JpaRepository에서

```
@Repository
```

해당 어노테이션을 사용하여 충돌한듯 하다.

이럴때는

### application.properties 에 

```
spring.main.allow-bean-definition-overriding=true
```

한 줄을 추가하여 준다.

![[Pasted image 20250221114644.png]]

성공적인 실행.

![[Pasted image 20250221114648.png]]

 이후 ' keys * ' 명령어를 사용하여 데이터가 정상적으로 추가된 것을 확인할 수 있다.