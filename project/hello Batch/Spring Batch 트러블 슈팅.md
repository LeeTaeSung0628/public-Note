
# 🙇‍♂Spring Batch 트러블 슈팅

#프로젝트 #개발 #SPRING #Batch #Partitioning #Chunk

---


# 작업 목적
기존 예치금 차액비교 Batch의 Tasklet방식의 배치의 단점을 보완하는 chunk 방식의 배치를 구현하고,
나아가 다른 기능의 Batch에도 효과적으로 빠르게 적용할 수 있는==재사용성/유지보수성 높은 코드==, 선례를 만들기 위함이다.

# 작업 계획
1. Chunk, Partioning방식을 Job을 추가 개발 (기존 balanceJob 유지)
2. 당분간 두 Job을 병행하면서 비교
3. 추가한 Job 기능에 문제없다면 기존 balanceJob 삭제

---

# 예치금차액비교 Batch Job 기존 소요시간 그래프

## 2024-10-28 ~ 2024-11-19 (주말제외)
- **평균 소요시간**: 약 17.35분
- **최대 소요시간**: 22분
- **최소 소요시간**: 12분
![[output (1).png]]

---

# 문제점 분석

## 1. 하나의 트렌젝션으로 동작하며, 실패시 처음부터 재시도 해야함
* *처음 가져온 Point 배치 완료시 까지 계속 물고있다.* -> ==한 트렌젝션의 범위가 넓다.==
## 2. 한 트렌젝션내에서 긴 시간을 동작하여, 배치중 일어나는 insert/update(예치금 입금 출금) 건을 감지하지 못함

---

# 배치 동작 로직

### 7:00 -> 회원 개개인 별 비교. 문자는 9시 30분에 예약문자로 오지만, 실제 로직은 7시에 돈다
- G5-Point 등은 처음 7시 시점에 묶여있다. 회원별로 ==실데이터를 건건이 api(신한)를 호출하여 비교한다. 때문에, 7시 이후에 수정된데이터를 실시간으로 반영하지 못한다.==
#### 신한API가 main데이터.

---

# New Batch의 주요 기술 및 목적
## Partitioning
- 목적 : batch의 step 레벨에서의 스레드 분리 ( 병렬처리 )
##### ** 프로세스(서비스 로직)단계에서 병렬처리를 하는 것과 어떠한차이가 있는가? ex) parallelStream, CompletableFuture **

| 항목           | **Partitioning (Spring Batch)**   | **ParallelStream, CompletableFuture** |
| ------------ | --------------------------------- | ------------------------------------- |
| **적용 계층**    | Batch Job의 Step 레벨                | 비즈니스 로직 또는 서비스 계층                     |
| **주요 목적**    | 대용량 데이터를 병렬 처리하여 Batch 속도 개선      | 연산 최적화, 비동기 처리, 응답 시간 단축              |
| **트랜잭션 관리**  | Spring Batch에서 제공 (트랜잭션 분리됨)      | 개발자가 직접 관리해야 함                        |
| **재시작 및 복구** | Spring Batch가 지원                  | 수동으로 구현 필요                            |
| **오버헤드**     | Spring Batch 컨텍스트 및 파티션 설정 오버헤드   | 상대적으로 가볍지만 트랜잭션 없음                    |
| **복잡도**      | 설정과 파티션 정의가 복잡함                   | 구현이 단순하고 직관적                          |
| **에러 핸들링**   | Spring Batch에서 관리                 | 개발자가 별도 핸들링 필요                        |
| **스레드 관리**   | Spring Batch의 PartitionHandler 관리 | ForkJoinPool 또는 스레드 풀 직접 관리           |
| **상태 관리**    | ExecutionContext를 통해 안전하게 관리      | 공유 변수나 상태 관리는 개발자 책임                  |
### 즉, 파티션 별로 독립적인 SlaveStep을 생성하기 떄문에 Spring Batch의 관리를 받을 수 있다.
- 독립적인 `ExecutionContext`가 주어져 상태를 안전하게 관리할 수 있다.


## Chunk
- 목적 : 각 step내의 트렌젝션 단위 분리
- Reader / Processor / writer 가 역할을 분담

---
# 파티셔닝 흐름

### ThreadPoolSize : 동시에 실행시킬 스테리드의 개수
### gridSize : 실제로 제단할 사이즈(작업단위)
### QueueCapacity : 대기열 크기
### MaxPoolSize : 최대 추가 스레드 풀 개수

##### ThreadPoolSize : 4  /  gridSize : 25 
##### MaxPoolSize : 4 / QueueCapacity : 2 (대기열크기를 벗어나면 Exception 발생)

#### 0. 250개의 데이터

4. **Partition 생성**: 
    
    - ==Partition 1== : ID 1 ~ 25 / ==대기열1== : ID 101 ~ 125 / ==대기열2== : ID 201 ~ 225
    - ==Partition 2== : ID 26 ~ 50 / ==대기열== : ID 126 ~ 150 / ==대기열2== : ID 226 ~ 250
    - ==Partition 3== : ID 51 ~ 75 / ==대기열== : ID 151 ~ 175 / ==대기열2== : -
    - ==Partition 4== : ID 76 ~ 100 / ==대기열== : ID 176 ~ 200 / ==대기열2== : -
    
5. **각 Partition에서 Chunk 처리**:
    
    - Partition 1:
        - Chunk 1: ID 1 ~ 10 → 커밋
        - Chunk 2: ID 11 ~ 20 → 커밋
        - Chunk 3: ID 21 ~ 25 → 커밋
    - Partition 2:
        - Chunk 1: ID 26 ~ 35 → 커밋
        - Chunk 2: ID 36 ~ 45 → 커밋
        - Chunk 3: ID 46 ~ 50 → 커밋
    - 나머지 Partition도 동일 방식으로 처리.
6. **병렬 실행**:
    - 스레드 풀 크기 = 4이므로 4개의 Partition이 동시에 실행됩니다.
    - Partition 처리 순서는 스레드 풀에서 처리되는 순서에 따라 다를 수 있음.
7. **트랜잭션 관리**:
    
    - 각 Partition은 독립적인 트랜잭션을 가짐.
    - 각 Chunk가 커밋될 때마다 트랜잭션이 종료됨.

---
### 데이터가 늘어났을때?

##### ThreadPoolSize : 4  /  gridSize : 25 
##### MaxPoolSize : 6 / QueueCapacity : 2 (대기열크기를 벗어나면 Exception 발생)
### 실행 흐름:

#### 0. 500개의 데이터

8. **Partition 생성**: 
    
    - ==Partition 1== : ID 1 ~ 25 / ==대기열1== : ID 101 ~ 125 / ==대기열2== : ID 201 ~ 225
    - ==Partition 2== : ID 26 ~ 50 / ==대기열== : ID 126 ~ 150 / ==대기열2== : ID 226 ~ 250
    - ==Partition 3== : ID 51 ~ 75 / ==대기열== : ID 151 ~ 175 / ==대기열2== : ID 251 ~ 275
    - ==Partition 4== : ID 76 ~ 100 / ==대기열== : ID 176 ~ 200 / ==대기열2== : ID 276 ~ 300
    - **추가 스레드 풀 생성**
    - ==Partition 5== : ID 300 ~ 325 / ==대기열== : ID 326 ~ 350 / ==대기열2== : ID 351 ~ 375
    - **추가 스레드 풀 생성**
    - ==Partition 6== : ID 376 ~ 400 / ==대기열== : ID 401 ~ 425 / ==대기열2== : ID 426 ~ 450
	
	**모든 대기열 소모 및 최대 스레드 풀 도달 => 작업 중단 및 오류**


---


# 파티셔닝 시 스레드 설정 방식 선택

### 어떠한 방식을 적용하는게 속도와 안정성 면에서 효율적일지??

## 처리할 컬럼 개수가 적을 때는 청크가 세분화되더라도 부하가 크지 않음.
-> 이로볼때, 쿼리작업보다 api호출작업의 소요시간이 길어보임

##### SimpleAsyncTaskExecutor vs ThreadPoolTaskExecutor

| **특징**        | **SimpleAsyncTaskExecutor** | **ThreadPoolTaskExecutor**        |
| ------------- | --------------------------- | --------------------------------- |
| **스레드 생성 방식** | 작업(파티션 단위)마다 새 스레드 생성       | 스레드 풀에서 스레드를 재사용                  |
| **스레드 개수 제한** | 없음                          | `corePoolSize`, `maxPoolSize`로 제한 |
| **대기열 지원**    | 없음                          | 작업 대기열(`queueCapacity`) 지원        |
| **대량 데이터 처리** | 비효율적                        | 효율적                               |
| **스레드 관리**    | 없음                          | 스레드 풀로 관리                         |
| **오버헤드**      | 스레드 생성/소멸로 오버헤드 큼           | 스레드 재사용으로 오버헤드 적음                 |
| **추천 사용 사례**  | 간단한 작업, 테스트용                | 대량 작업, 병렬 처리, 안정적 처리 필요 시         |

___

범위 - 2017-11-07 ~ 2017-12-01
startDate=2017-11-07&endDate=2017-12-01
## 컬럼 개수 - 292개

기존 병렬처리 : 1분 7초
기존로직 : 6분 32초

t:스레드 개수 / c:각 스레드별 청크 파티션 수

###### SimpleAsyncTaskExecutor vs ThreadPoolTaskExecutor

SimpleAsyncTaskExecutor
g4/c10 : 1분 41.586초
g5/c10 : 1분 30.037초
g5/c3 : 1분 28.997초
g5/c1 : 1분 29.931초
g8/c10 : 1분 36.019초
g10/c1 : 1분 24.529초

-

ThreadPoolTaskExecutor
Th min/maxSize64, g64/c10 : 1분 42.848초
Th min/maxSize32, g32/c10 : 1분 26.551초

Th min/maxSize16, g16/c10 : 
1분 46.753초 / 1분 33.779초 / 2분 25.759초
1분 11.668초 / 2분 5.773초 / 2분 39.553초

Th min/maxSize32, g8/c20 : 3분 25.743초

Th min/maxSize16, g4/c10 :  2분 29.207초 / 2분 8.004초

### 정리
#### SimpleAsyncTaskExecutor 
- Grid사이즈(파티셔닝 갯수) 만큼 스레드 풀 계속 생성
- 파티셔닝 갯수가 많아졌을 때, 스레드 생성/소멸에 드는 오버헤드 증가
#### ThreadPoolTaskExecutor
- 최소 스레드풀 개수 / 최대 스레드풀 개수 / 대기열 큐 크기 / 유휴 시간 모두 설정가능
- 파티셔닝 갯수가 많을 경우, 처리 성능에 맞는 스레드 풀에서 대기작업으로 처리하여 오버헤드 감소

### 1. 예치금 차액 처리 건은 총 컬럼수가 많지 않다. 
### 2. 배치 프로젝트 특성상 동시 작업을 고려하지 않아도 된다.
- 총 처리량 기준 1/n로 각 스레드에 할당시 충분히 감당한 양으로, 대기열을 사용할 필요가 없다.
## 따라서, ==SimpleAsyncTaskExecutor를== 통해 관리하는 것이 적합하다고 판단하였다.

---

# 청크 방식의 정확한 동작 로직
## 1. ==리더==는 청크사이즈 만큼 반복하며, 각 반복마다 특정범위의 값을 return한다.
## 2. ==프로세서==는 청크사이즈와 관계없이 리더가 넘긴 return에 따라 동작을 수행한다. 
## 3. ==롸이터==는 청크사이즈만큼 모이면, 그때 1번 동작한다.


---
# QuerydslPagingItemReader 적용기
## ItemReader 방식
- 쿼리 호출 및 페이징 기법 직접 구현

## PagingItemReader 방식
- Native Query 형태로 호출하여 자동 페이징

## QuerydslPagingItemReader (커스텀) 방식
- Spring Batch에서는 공식적으로 지원하지 않음
- 자동 페이징
- ==Querydsl==형태로 쿼리 호출 가능

### QuerydslPagingItemReader 방식 구현 목적
- 기존(헬로)의 Querydsl기반 JPAQuery를 그대로 사용 가능
- 자동 페이징
- 그 외 Querydsl의 장점 보유

### AbstractPagingItemReader를 상속받는 커스텀 QuerydslPagingItemReader 클래스 생성
- 기존 PagingItemReader의 메서드를 오버라이딩 하여 사용하며, 
 JPQL이 수행되던 부분에 코드를 수정하였다.

#### 구현된 Reader
``` java
@Bean  
@Scope(value = "step", proxyMode = ScopedProxyMode.TARGET_CLASS)  
public QuerydslPagingItemReader<HfbatBankBalanceCheckDto> balanceReader() {  
    ExecutionContext jobContext = Objects.requireNonNull(StepSynchronizationManager.getContext()).getStepExecution().getJobExecution().getExecutionContext();  
    Date startDate = (Date) jobContext.get(START_DATE_KEY);  
    Date endDate = (Date) jobContext.get(END_DATE_KEY);  
    return new QuerydslPagingItemReader<>(  
            entityManagerFactory,  
            executionOrder,  
            DEFAULT_CHUNK_SIZE,  
            queryFactory -> repository.newFindChangeBalanceMemberList(  
                    startDate,  
                    endDate  
            ));  
}
```


---

# QuerydslPagingItemReader를 사용하며, 다중스레드 스케줄링 하기

## 목적
- 각 파티션 스레드 별 종료 시간이 크게 상이하다. ==총 소요시간 기준 최대 약 20% 차이==
- 적용 한다면, 각 스레드 별로 even하게 작업을 수행하여 총 소요시간을 줄일 수 있을것이라 판단 

## 구현 방법
``` java
JOB

private static AtomicLong executionOrder = new AtomicLong(0);

...

new QuerydslPagingItemReader<>(  
        entityManagerFactory,  
        executionOrder,  
        DEFAULT_CHUNK_SIZE,  
        queryFactory -> repository.newFindChangeBalanceMemberList(  
                startDate,  
                endDate  
        ));
```
- job 레벨에서의 스레드 세이프한 전역 변수를 생성하였다.
- 그 후, 각 파티션 별 리더에 인자값으로 넘긴다.

```
long currentExecutionOrder = executionOrder.getAndIncrement();  
long startIndex = (currentExecutionOrder) * getPageSize();  
int totalRecords = stepContext.getInt("totalRecords");  
  
if (startIndex >= totalRecords) {  
    initResults(); // 빈 결과로 초기화  
    tx.commit();  
    return;  
}  
  
int chunkSizeToRead = Math.min(getPageSize(), (int) (totalRecords - startIndex)); // 남은 데이터 크기만큼 읽기  
  
// QueryDSL Query 생성  
JPQLQuery<T> query = createQuery()  
        .offset(startIndex)  
        .limit(chunkSizeToRead);
```
### 스레드 세이프한 해당 변수는 각 페이징 리더의 startIndex를 지정 한 후,
### 즉시 값을 늘린다. getAndIncrement()


## 효과
- 먼저 한 청크단위의 작업을 끝낸 파티션은 그 다음 작업을 즉시 할당받게 된다.
- 즉, 총 처리시간 기준 ==가장 빨리끝난 파티션==과 ==가장 늦게 끝난 파티션==의 실행 시간 차이는 최대 한 청크사이즈를 처리하는 시간보다 크지 않게 된다.

---

# 메세징 처리 로직 분리

## 1. 로직순서

### 메세징 처리할 데이터
1차 비교 이후 차액이 발생한 녀석들의 List 를 한번 더 검증한 후,  => 추후 변경 됨
검증된 녀석들을 
``` java
List<BalanceCheckResultDto> realDiffList= new ArrayList<>();
```
최종적인 차액 리스트에 넣는다.

### 스레드 별로 따로 처리되는 녀석들을 processor의 리턴으로 writer에게 넘긴다.
writer는 각 프로세서의 리턴으로 받은 녀석들을 하나의 DTO List로 합쳐 메세징 처리를 하게 된다.

---

## 2. 어떻게 메세지(DTO 데이터)를 취합할 것인가?
##### 전역으로 객체를 선언하여 append하는 방식은 스레드 세이프 하지 않기 때문에 배제하였다.
### 방법 1. 외부저장소를 사용한다.
- 유지보수성이 좋다.
- 환경셋팅에 리소스가 많이든다. 
### 방법 2. ExecutionContext에 DTO를 직렬화 시켜 저장한다.
- 구현 레벨이 가장 쉽다
- DTO(데이터)가 많아질 경우, 큰 리소스를 차지한다.
### 방법 3. DTO를 분해하여 기본 데이터만 저장한다.
- 방법 2.보다는 리소스가 적지만, 문자열 처리 시간이 추가로 소모된다.
### 방법 4. ConcurrentLinkedQueue 사용 (스레드 안전한 큐 자료구조)
- 높은 쓰기 성능을 갖고있다.
- 동시성 문제 없이 다중 스레드에서 사용 가능하다.
- 인덱스로 접근이 불가능하다.

#### 해당 방법들을 현재 프로젝트와 비교했을 때, 방법 4.가 합리적으로 보인다.

QueueManager클래스를 생성하며 공통으로 사용 가능하도록 하였고,
``` java
@Slf4j  
public class QueueManager<T> {  
    protected final ConcurrentLinkedQueue<T> sharedQueue = new ConcurrentLinkedQueue<>();  
  
    // 데이터 추가  
    public void addItemToSharedQueue(T item) {  
        if (item != null) {  
            sharedQueue.add(item);  
        }  
    }
```

이를 상속받아 특정 오브젝트를 넘길 수 있도록 하였다.
``` java
public class BalanceQueue extends QueueManager<BalanceCheckResultDto>{  
  
    public List<BalanceCheckResultDto> getDtoFromQueue() {  
        return super.getItemsFromQueue();  
    }  
}
```

---

# **시행착오**

# Limit을 사용하지 않고 JpaPagingReader를 사용했을 때, 프로세서가 모아서 처리할 수 있는 방법이 있는가?

## 이게 정확히 무슨소리인가?
- 보통의 Batch 서비스라면 I/O 작업에 부하가 걸려있겠지만, 현재 예치금 차액 배치는 processor 즉, 예치금 비교 연산에서 큰 리소스를 소모하고있다.
- 이에 해당하는 시간 소모를 줄이기 위해 processor(서비스로직) 을 병렬처리 함으로 최종 처리시간 단축을 꾀할 수 있을지에 대한 고민이다.
- *이후 설명하겠지만, 내부 병렬처리 로직은 청크의 트렌젝션을 무너뜨릴 가능성이 크므로 지양해야함.*



### - ==processor==는 ==reader==의 return결과로만 트리거 되기 때문에 방법이 없다.
![[Pasted image 20241206113120.png]]
## 필수가 아니라면, ==processor==의 로직을 ==writer==에 할당하여 청크사이즈로 컨트롤 할수 있지 않을까?
#### reader(dto) -> ~~processor~~ -> writer(dtoList)

##### - 구현 결과 ==reader==의 return인 dto값을 chunk사이즈 만큼 List dto로 합쳐 일괄 처리가 가능하다.

	소요시간 : 1분 9초
-> 기존 limit절을 이용한 로직보다 속도가 더빠르며,
   JpaPagingReader를 적용한다면 중복 select횟수를 줄여 더 빨라질 것으로 예상된다.

### 우려되는 점은 writer가 데이터 가공의 역할까지 맡는게, 설계 목적에 부합한지 생각해보아야 한다.

결론 : 둘 중 하나
##### 1. 리더,프로세서 각각 dto 1개 씩 처리 (역할 분담)
##### 2. 프로세서 로직을 제거하여 스레드는 유지하돼, 서비스로직에서 dto list로 내부 병렬처리 로직으로 구현하기

```
#### 292 컬럼
## Processor 삭제(병렬처리)로직
#### grid-size:6 / chunk-size:20
54679 ms
54416 ms
## Processor 순차처리(writer만 병렬처리) 로직

#### grid-size:6 / chunk-size:20
60440 ms
62271 ms
#### grid-size:10 / chunk-size:20
58129 ms
56723 ms
#### grid-size:16 / chunk-size:20
58595 ms
56314 ms

#### grid-size:32 / chunk-size:20
- SQLTransientConnectionException
- 스레드풀 점유갯수 초과
```

## 데이터 상으로는 내부 병렬처리의 승리
- 하지만, 이후 청크 방식 Batch의 확장성과 유지보수성을 고려하여 ==리더,프로세서,라이터== 방식으로 구현하기로 정했다.

---

# 청크사이즈가 다름에도 처리속도가 똑같은 이유?

### 예상 - 트렌젝션이 분리가 안됬나?

```
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

### 원인이 무엇일까?

- 트렌젝션 분리가 되어있지 않았기 떄문에 청크사이즈를 아무리 잘게 쪼개도 시간이 같았던 것이다.
- 그럼 트렌젝션이 분리되지 않은 이유는?

```
dtoList.stream().parallel()  
        .forEach(dto -> {});
```

parallel문으로 내부 병렬처리로직을 구현했기 때문에, chunk의 트렌젝션에서 벗어나,
매 insert문 마다 커밋을 하였던 것이다.

## 트렌젝션 관리시 병렬처리 로직의 사용에 주의할것.

---

# 반복 TEST 중.. 커넥션풀 Time Out 문제??

### DB 커넥션 풀 개수 확인
SHOW VARIABLES LIKE 'max_connections'; //최대 개수
SHOW STATUS LIKE 'Threads_connected'; //사용중인 개수


![[Pasted image 20241213102128.png]]
## 처음 몇 번간은 정상실행 되지만, 반복 테스트 중 스레드 풀 점유 대기 타임아웃이 발생했다.

### 원인은 무엇인가??

- 배치를 완료한 이후에도 스레드 풀(히카리 풀)을 반환하지 않는지 확인
### 배치를 반복할 수록 커넥션 엑티브된 ==커넥션 풀 개수가 점점 늘어나는== 모습
![[Pasted image 20241213102229.png]]

## 원인?

- Step이 마무리될때, 적어도 Job이 마무리 될때 ==entityManager==를 클로즈 시키는것이 자명한데, 어째서 커넥션풀이 해제되지 않는가??

### 앤티티 매니저가 클로즈 되었음에도 커넥션풀을 물고있는 모습.
![[Pasted image 20241213143101.png]]

## *** **엔티티 매니저는 클로즈 될 때, 트렌젝션이 살아있다면 그 트렌젝션이 종료될때까지 기다린다**
![[Pasted image 20241213143217.png]]

## 수정 후

### 정상적으로 커넥션 풀이 점유 해제된 모습
![[Pasted image 20241213103503.png]]


---

# Listener 역할 분리하고, 재사용 가능하도록 만들자


### * Listener에 포함된 초기화 코드 및 로그 로직
![[Pasted image 20241217150339.png]]
- 리스너에 너무 많은 역할이 부여된 모습
#### 파티션 핸들러의 grid사이즈를 동적으로 할당하기 위해서 필요한 작업.


## 파티션 핸들러의 생성 시점을 명확히 한 후에, 리스너에서 주입하던 코드를 STEP으로 분리하였다.
![[Pasted image 20241219145746.png]]
##### 기존 SteoExecution에서 JobExecution으로 하나의 Job에서 공유할 수 있도록 변경하여 gridSize를 다음 Step에서 가져올수 있도록 하였다.

### 비교: JobParameters vs ExecutionContext`

| 특징                  | `JobParameters`    | `ExecutionContext`                   |
| ------------------- | ------------------ | ------------------------------------ |
| **읽기/쓰기 여부**        | 읽기 전용              | 읽기/쓰기 가능                             |
| **공유 범위**           | Job 전체에서 공유        | Step 또는 Job 전체에서 공유 가능               |
| **용도**              | 실행 시 매개변수 전달       | 실행 중 상태 저장, 데이터 공유                   |
| **수명**              | JobInstance와 함께 유지 | StepExecution 또는 JobExecution과 함께 유지 |
| **Restart 시 유지 여부** | 항상 유지됨             | Restart 가능한 상태만 유지                   |

#### StepExecution vs JobExecution

- **StepExecution-Level ExecutionContext**
    
    - 각 Step에 고유한 `ExecutionContext`가 생성됩니다.
    - **Step 내의 Reader, Processor, Writer 등에서 공유**됩니다.
    - 다른 Step과는 공유되지 않습니다.
    
    - `balanceWorkerStep` 내에서는 Reader, Processor, Writer가 동일한 `ExecutionContext`를 공유합니다.
    - `balancePartitionStep`과 `balanceWorkerStep`의 `ExecutionContext`는 서로 독립적입니다.
- **JobExecution-Level ExecutionContext**
    
    - **Job 전체에서 공유**되며, 모든 Step이 동일한 `ExecutionContext`에 접근할 수 있습니다.
    - `JobExecutionContext`는 Step 간 데이터 전달이 필요할 때 유용합니다.

## 역할이 분리된 리스너는 재사용을 위해 커스텀 클래스로 생성

JobExecutionListener 객체 생성
```
@Slf4j  
public class JobTimerExecutionListener implements JobExecutionListener {  
    private final String jobName;  
    private long startTime = System.currentTimeMillis();  
  
    public JobTimerExecutionListener(String jobName) {  
        this.jobName = jobName;  
    }  
  
    @Override  
    public void beforeJob(JobExecution var1) {  
        startTime = System.currentTimeMillis();  
    }  
  
    @Override  
    public void afterJob(JobExecution var1) {  
        long endTime = System.currentTimeMillis();  
        long elapsedTime = endTime - startTime;  
        long minutes = (elapsedTime / 1000) / 60; // 밀리초를 분으로 변환  
        double seconds = (elapsedTime / 1000.0) % 60; // 남은 밀리초를 초로 변환 (소수점 포함)  
        log.info("{}-completed: {} ms | {} minutes {} seconds", jobName, elapsedTime, minutes, seconds);  
    }  
}
```


---
# **예치금 배치 구조도**

## [[배치 리팩토링 구조 드로잉]]

---

# 성과

## Batch 총 소요시간 그래프

![[output (9) 1.png]]
#### 평균 총 소요시간
##### 기존로직  : 13.27분
##### 신규로직 : 4.77분

## 1컬럼당 소요시간 그래프
![[output (10).png]]
#### 1컬럼 당 소요시간 
##### 기존로직  : 3.74초
##### 신규로직 : 0.72초

---

###### grid-size : 6 기준
### 기존로직 대비 
### ==총 소요시간== **64.05%** 감축
### ==1컬럼당 처리시간== **80.75%** 감축
### ==1트렌젝션 최대 처리시간== **99.91%** 감축

---

# Grid-Size에 따른 ==점유 커넥션 풀 및 소요시간==

![[do-messenger_screenshot_2024-12-23_14_40_21.png]]
#### grid-size를 동적으로 늘려감에 따라 ==총 소요시간을 더욱 줄일 수 있다==.

---

# API 부하 검증 Elastic APM

![[Pasted image 20241224093159.png]]
![[Pasted image 20241224093207.png]]
![[Pasted image 20241224093225.png]]

---

# + 플로우 차트 추가

![[회사 은행 서비스1111.png]]
![[Spring Batch Tasklet 예치금 잔액 비교 1.png]]
![[Spring Batch Tasklet 예치금 잔액 비교 잘못된 차액발생 인식 타임라인.png]]


[[Batch - 예치금차액내역 리펙토링 부록]]
