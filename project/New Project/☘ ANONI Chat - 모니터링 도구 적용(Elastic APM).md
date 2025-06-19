# ☘ ANONI Chat - 모니터링 도구 적용(Elastic APM)

#프로젝트 #개발 #보안 #인프라 #HTTPS #트러블슈팅 #Elasticsearch #APM #모니터링

---

<br>

#### 지난 포스트
#### ▶ [[☘ ANONI Chat - infra setup]]
#### ▶ [[☘ ANONI Chat - CICD 구성]]
#### ▶ [[☘ ANONI Chat - ELK Stack setting]]
#### ▶ [[☘ ANONI Chat - NGINX(feat. Kibana오류와 HTTPS 적용하기)]]


---

<br>

# <font color="#76923c">개요</font>

- 기존의 셋팅된 elK-stack에 elastic APM을 적용해 보도록하겠다.

<br>

---
<br>

# <font color="#8db3e2">Elastic APM이 무엇이며, 왜 적용했는가?</font>

<br>

## APM이란?
- APM은 **A**pplication **p**erformance **M**onitoring 의 약자이다.
- 애플리케이션의 **성능정보** 및 **발생한 로그정보**, 동작중인 **서버의 Metric정보**를 수집한다.
- MSA환경에서 서비스를 구성하는 여러 앱간의 Request를 하나의 Trace로 묶어서 추적할 수 있다.(**분산 Tracing**)
- Application 지연이 발생했을 때, 지연에 대한 병목 구간을 찾아낼 수있는 모니터링 서비스이다.

>[!info] Metric 정보란?
> 시간이 지남에 따라 변화하는 데이터를 의미한다.
> 메모리 사용률, CPU 사용률, 스레드 사용률 등 **시간에 따른 추이를 추적할 가치가 있는 데이터**이다.
> 
> 어떤 서비스, 앱이냐에 따라 해석이 달라질 수는 있지만, 보편적으로 대시보드를 볼때 특정 수치들을 그래프로 보여주는 일종의 **시각화**로 이해하기도 한다.


<br>

