```yml
version: '3' 

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
    networks:
      - elk
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
	  image: docker.elastic.co/logstash/logstash:7.12.0
	  ports:
	    - "5044:5044"
	    - "5000:5000"
	  volumes:
	    - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
	  networks:
	    - elk

  kibana:
    image: docker.elastic.co/kibana/kibana:7.11.1
    ports:
      - "5601:5601"
    networks:
      - elk
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - server.host=0.0.0.0

  spring:
    image: ghcr.io/anonichat/app/anonichat
    ports:
      - "8081:8080"
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - elk

volumes:
  esdata:
    driver: local

networks:
  elk:
    driver: bridge
```