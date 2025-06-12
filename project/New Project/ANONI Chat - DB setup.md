# ANONI Chat - DB setup

#프로젝트 #개발 #보안 #인프라 #HTTPS #트러블슈팅 #DB #mysql

---

<br>

# <font color="#76923c">개요</font>

- Nginx 적용 전, DB셋팅을 먼저 진행하려고 한다.


---

<br>

## Spring 셋팅

application.properties

```java
# DataSource configuration for MySQL  
spring.datasource.url=${SPRING_DATASOURCE_URL}  
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}  
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}  
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver  
  
# JPA settings  
spring.jpa.hibernate.ddl-auto=update  
spring.jpa.show-sql=true  
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

- `${SPRING_DATASOURCE_**}` 처리 되어있는 변수는 보안과 일관성을 위해 DockerCompose셋팅시 넘겨주는 매개변수 값이다.

<br>

## DockerCompose(elk) 셋팅

```python
... elk 는 생략

  spring:
    image: ghcr.io/anonichat/app/anonichat
    ports:
      - "80:8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/anonichat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul
      - SPRING_DATASOURCE_USERNAME=anonichat
      - SPRING_DATASOURCE_PASSWORD=hf134679!@#
    depends_on:
      - elasticsearch
      - mysql
    networks:
      - elk  # 기존 elk stack 네트워크
      - data # Spring, Mysql, Jenkins를 묶어줄 네트워크

  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=anonichat
      - MYSQL_ROOT_PASSWORD= 'root비밀번호'
      - MYSQL_USER= 'db 유저이름'
      - MYSQL_PASSWORD= 'db 비밀번호'
    volumes:
      - mysql_data:/home/hello/Desktop/AnoniChat/elk-stack/data/mysql
    networks:
      - elk  # 기존 elk stack 네트워크
      - data # Spring, Mysql, Jenkins를 묶어줄 네트워크
networks:
  elk:
  data:

volumes:
  esdata:
  mysql_data:
```

- spring컨테이너와 mysql컨테이너를 `data`네트워크로 묶어준다.

<br>

## DockerCompose(jenkins) 셋팅

```python
docker volume ls | grep jenkins_home

version: '3.8'

services:
  jenkins:
    image: ghcr.io/anonichat/app/jenkins-dood:v0.07
    container_name: jenkins-dood
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/home/hello
      - /home/hello/Desktop/AnoniChat/elk-stack:/home/hello/Desktop/AnoniChat/elk-stack
    networks:
      - data

networks:
  data:
    driver: bridge

volumes:
  jenkins_home:
    external: true

```

- 동일핳게 `data` 네트워크로 생성.

>[!warning] 주의
> 같은 네트워크로 묶는다 하여도, Jenkins를 위한 docker-compose파일이 다른 디렉토리에 위치한다면
> 다른 네트워크를 갖게된다.
>
 명시적으로 프로젝트명을 지정하지 않으면, docker-compose파일이 위치한 디렉토리명을 *프로젝트 네임*으로 갖게되기 때문.
> 따라서, `docker-compose -p elk-stack up -d`와 같이 기존 컨테이너들과 *프로젝트명을 동일*하게 가져가야한다.

![[Pasted image 20250612135635.png|525]]
<br>

## 실행결과

#### DB추가 후 CI/CD
![[Pasted image 20250612143924.png]]

#### DB데이터 select Test
![[do-messenger_screenshot_2025-06-12_14_47_01 1.png]]
![[do-messenger_screenshot_2025-06-12_14_47_40.png]]