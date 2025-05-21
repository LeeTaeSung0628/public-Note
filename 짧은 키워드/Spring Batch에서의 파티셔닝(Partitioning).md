# Spring Batch에서의 파티셔닝(Partitioning)
- Tasklet 방식과 Chunk 방식 모두 사용 가능하지만, 보통 Chunk방식에서 사용한다.
### 파티셔닝(Partitioning)
- 대량 데이터를 효율적으로 처리하기 위해 데이터를 여러 개의 작은 조각(Partition)으로 나눠 **병렬적으로 처리**하는 기술입니다. Spring Batch에서 파티셔닝은 **마스터-슬레이브 패턴**을 사용하며, 마스터는 작업을 분할하고 슬레이브는 각각의 분할된 작업을 수행합니다.

#### 파티션너(파티션 별 범위 - 컨텍스트 범위 설정)
``` java
import org.springframework.batch.core.partition.support.Partitioner;
import org.springframework.batch.item.ExecutionContext;
import java.util.HashMap;
import java.util.Map;

public class RangePartitioner implements Partitioner {

    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        Map<String, ExecutionContext> partitions = new HashMap<>();
        int min = 1; // 데이터베이스 ID의 최소값
        int max = 5; // 데이터베이스 ID의 최대값
        int targetSize = (max - min) / gridSize + 1; // 각 Partition의 범위 크기

        int start = min;
        int end = start + targetSize - 1;

        for (int i = 0; i < gridSize; i++) {
            ExecutionContext context = new ExecutionContext();
            context.putInt("minId", start); // 시작 ID
            context.putInt("maxId", end); // 종료 ID
            partitions.put("partition" + i, context);

            start += targetSize;
            end += targetSize;
        }

        return partitions;
    }
}
```

### **Chunk와 Partitioning의 병렬 실행 예제**

#### 예제 시나리오:

- 데이터베이스에 100개의 레코드가 있음.
- `gridSize = 4`: 데이터를 4개의 Partition으로 나눔.
- `chunkSize = 10`: 각 Partition에서 데이터를 10개씩 읽어 처리.
- 스레드 풀 크기 = 4: 4개의 Partition이 동시에 실행 가능.

---
코드가 여러줄 나올거같진 않아서 방식은 
페어 프로그래밍 방식으로 진행하도록 하죠

개발 절차는 이렇게 갈 예정이에요

4. Chunk, Partioning방식을 Job을 추가 개발 (기존 balanceJob 유지)
5. 당분간 두 Job을 병행하면서 비교
6. 추가한 Job 기능에 문제없다면 기존 balanceJob 삭제

#### 실행 흐름:

7. **Partition 생성**:
    
    - Partition 1: ID 1 ~ 25
    - Partition 2: ID 26 ~ 50
    - Partition 3: ID 51 ~ 75
    - Partition 4: ID 76 ~ 100
8. **각 Partition에서 Chunk 처리**:
    
    - Partition 1:
        - Chunk 1: ID 1 ~ 10 → 커밋
        - Chunk 2: ID 11 ~ 20 → 커밋
        - Chunk 3: ID 21 ~ 25 → 커밋
    - Partition 2:
        - Chunk 1: ID 26 ~ 35 → 커밋
        - Chunk 2: ID 36 ~ 45 → 커밋
        - Chunk 3: ID 46 ~ 50 → 커밋
    - 나머지 Partition도 동일 방식으로 처리.
9. **병렬 실행**:
    
    - 스레드 풀 크기 = 4이므로 4개의 Partition이 동시에 실행됩니다.
    - Partition 처리 순서는 스레드 풀에서 처리되는 순서에 따라 다를 수 있음.
10. **트랜잭션 관리**:
    
    - 각 Partition은 독립적인 트랜잭션을 가짐.
    - 각 Chunk가 커밋될 때마다 트랜잭션이 종료됨.

### ThreadPoolSize : 동시에 실행시킬 스테리드의 개수
### gridSize : 실제로 제단할 사이즈(데이터를 얼마나 세분화해서 각 파티션 작업에 할당할 건지)
### QueueCapacity : 대기열 크기

---