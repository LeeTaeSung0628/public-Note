# Kibana오류와 HTTPS 적용하기

#프로젝트 #개발 #보안 #인프라 #HTTPS #트러블슈팅

---

<br>

# <font color="#76923c">개요</font>

- elasticsearch 라이센스 및 보안 문제

<u>Kibana에 접속시, UI가 안보이고 오류로그가 JSON 형태로만 보이는 오류가 발생.</u>

![[do-messenger_screenshot_2025-06-09_17_27_36.png|600]]

찾아보니 elasticsearch 7.11+ 버전에서 발생하는 문제라고 한다.

<br>

우선 임시방편으로 docker-compose.yml을 수정할 필요가 있다.  
<u>라이센스를 basic으로 명시해줘야하고 보안 설정을 false로 해야한다.</u>

보안 설정을 해제하는 이유는 보안 설정을 하면 HTTPS 사용이 강제되어서 HTTP로는 접근이 불가능하기 때문이다.  
>[!info] 정보
>현재는 환경 구축 단계이고 **HTTPS**를 적용하지 않은 상태이기 때문에 나중에 **HTTPS** 설정을 하고 보안 설정을 다시 할 예정이다.

docker-compose.yml 변경 사항

```yml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=false     # 보안 기능 비활성화
    - xpack.license.self_generated.type=basic  # 기본 라이센스 사용
  ports:
    - "9200:9200"
  networks:
    - elk
  volumes:
    - esdata:/usr/share/elasticsearch/data

kibana:
  image: docker.elastic.co/kibana/kibana:7.11.1
  ports:
    - "5601:5601"
  networks:
    - elk
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    - server.host=0.0.0.0
    - xpack.security.enabled=false     # Kibana 보안 비활성화
    - xpack.license.self_generated.type=basic
```

---

<br>

# <font color="#76923c">HTTPS 적용하기</font>

<br>

## Spring