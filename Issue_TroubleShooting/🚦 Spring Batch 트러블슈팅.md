# 🚦 Spring Batch 트러블슈팅

#트러블슈팅 #EntityManager #Transaction #트렌젝션

---
# 프로젝트 보러가기 ◀
### [[🏹 예치금 차액 비교 Spring Batch 리펙토링]]

---

# <font color="#9bbb59">첫 번째 이슈</font>

>[!error] 청크사이즈가 다름에도 처리속도가 똑같은 이유가 뭘까?

** Chunk 방식의 Batch에서 ChunkSize란, 한 트렌젝션 내에서 처리할 컬럼(DTO/모델)의 개수이다.

즉, ChunkSize가 작을수록 *데이터 I/O작업* 및 *Overhead*(데이터 읽기/쓰기, 트랜잭션 시작 및 종료 등)가 *증가하여* 총 실행시간이 *길어져야한다*.

- 청크 사이즈별 실행시간 측정 데이터
```json
grid-size:12 / chunk-size:30
-
3분 22.856초
3분 23.096초

grid-size:12 / chunk-size:20
-
3분 23.546초
3분 23.784초


grid-size:12 / chunk-size:10
-
3분 24.243초
3분 22.389초
3분 24.667초
3분 24.789초

grid-size:12 / chunk-size:5
-
3분 24.953초
3분 24.353초
```
# ▶ *유의미한 차이를 보기어렵다.*

---

## <font color="#9bbb59">트러블 슈팅</font>

#### 예상 - 트렌젝션이 분리가 정상적으로 되지 않았나?
### 원인이 무엇일까?

- 배치 서비스 로직의 일부
```java
dtoList.stream().parallel()  
        .forEach(dto -> {});
```

- 스레드 확인
```java
}
	}
        IntStream.range(1, 10)
                .parallel()
                .forEach(i -> System.out.println(Thread.currentThread().getName() + " - " + i));
    }
}

- 출력
ForkJoinPool.commonPool-worker-3 - 3
ForkJoinPool.commonPool-worker-1 - 1
ForkJoinPool.commonPool-worker-2 - 2
ForkJoinPool.commonPool-worker-0 - 4
...

```

---

## 해당 코드가 문제가 될까?

- 별도의 셋팅이 없이 `parallel()` 구문을 사용하여 병렬 처리를 진행한다면, 
1.**데이터를 작은 단위(Chunk)로 분할**
    - 예를 들어, 1000개의 데이터를 4개의 스레드에서 처리한다고 하면, `ForkJoinPool`은 데이터를 여러 개의 Task로 나눈다.
2.**Worker Thread들이 분할된 작업을 병렬로 실행**
    - 각 스레드는 자신이 맡은 작업을 처리하고, 남는 작업이 있다면 다른 스레드의 작업을 훔쳐(Work-Stealing) 가져와 실행한다.
3.**최종적으로 결과를 합쳐서 반환**
    - 모든 작업이 완료되면 병렬 처리된 결과가 하나로 합쳐진다
    - 
>[!note] `ForkJoinPool`이란?
> Java에서 제공하는 **병렬 작업을 최적화하는 스레드 풀**로써,
> **Work-Stealing 알고리즘**을 사용하여 유휴스레드를 최소화하고 CPU 활용도를 극대화하는 기법이다.

해당 과정에서 `ForkJoinPool`은 작업을 여러 개의 *워커 스레드*(`ForkJoinPool-worker-*`)에서 실행하기 때문에 **Spring의 ThreadLocal 기반 트랜잭션이 전파되지 않는다.**

---

# 결론
- 청크의 내부 서비스로직에서의 동작 효율을 위해 병렬처리를 사용했으나, `ForkJoinPool`기반의 정확한 동작원리를 충분히 고려하지 않아 발생한 이슈이다.
- 해당 부분은 일반적인 `forEach`문으로 변경함으로써 이슈를 해결할 수 있었다.

*특히, 멀티스레드를 다룰 때에는 병렬처리를 함에 있어 주의를 필요로 함을 깨달았다.*

---
---

# <font color="#9bbb59">두 번째 이슈</font>

>[!error] 반복 TEST 중.. 커넥션풀 Time Out 문제??

![[Pasted image 20241213102128.png]]
### DB 커넥션 풀 개수 확인
```sql
SHOW VARIABLES LIKE 'max_connections'; //최대 개수
SHOW STATUS LIKE 'Threads_connected'; //사용중인 개수
```
![[Pasted image 20250304170347.png]]
![[Pasted image 20250304170412.png]]
![[Pasted image 20250304171240.png]]
- 운영 DB의 커넥션 pool은 충분한 상태로 보인다. 

---

>[!info] 정보
> 처음 몇 번간은 정상실행 되지만, 반복 테스트 중 스레드 풀 점유 대기 타임아웃이 발생했다.

- 배치를 완료한 이후에, 스레드 풀(히카리 풀)을 정상 반환하는지 확인.
#### 배치를 반복할 수록 커넥션 엑티브된 *커넥션 풀 개수가 점점 늘어나는* 모습
![[Pasted image 20241213102229.png|750]]

---
## 원인?

먼저, **QuerydslPagingItemReader**의 doReadPage()의 **종료조건**에서 트렌젝션 **커밋**을 별도로 수행하지 않고 리턴을 시키고 있었다.
#### 클라이언트 연결이 종료(connection clos)가 일어나면 트렌젝션도 종료되지 않는가??

-> 그럼에도 Step이 마무리될때, 적어도 Job이 마무리 될때, **entityManager**를 클로즈 시키는것이 자명한데, 어째서 커넥션풀이 해제되지 않을 수 있는가.

 - 앤티티 매니저가 클로즈 되었음에도 커넥션풀을 물고있는 모습.
![[Pasted image 20241213143101.png]]
- 코드를 뜯어보자...

먼저, 앤티티 매니저는 커넥션의 반환을 위임받는다.
![[Pasted image 20250314120439.png]]
그리고.. transactionObserver는 앤티티 매니저 객체와 연결된 트랜젝션 매니저가 종료되기를 바라보고 있었다.

즉,
>[!danger] 주의
>엔티티 매니저는 클로즈 될 때, 트렌젝션이 살아있다면 그 트렌젝션이 종료될때까지 기다린다.
>
![[Pasted image 20241213143217.png]]

---

## 결론

- 엔티티 매니저가 **클로즈** 될 떄, **커넥션 pool이 반환될거라는 기대와는 달리**,
	트렌젝션이 커밋 될 때 까지 커넥션pool을 계속해서 물고 있었다.
	(reader는 페이징을 위한 별도의 트렌젝션 생성)

#### **단순 조회(select)쿼리**는 트렌젝션을 중요하게 생각하지 않고 설계를 해왔지만, 이번 커넥션Pool 점유 이슈에서 트렌젝션 설계의 중요성을 다시금 깨달았다.

---

## 수정 후
```java
protected void doReadPage() {  
	...
        if (startIndex >= totalRecords) {  //마지막인덱스 확인
            initResults(); // 빈 결과로 초기화  
            tx.commit();  //트렌젝션 커밋
            return;  
        }  
	...
```

### 정상적으로 커넥션 풀이 점유 해제된 모습
![[Pasted image 20241213103503.png]]
