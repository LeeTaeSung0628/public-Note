# 📓 EntityManager와 Transaction 그리고 ConnectionPool

#트러블슈팅 #EntityManager #Transaction #트렌젝션

---

# 트러블 슈팅


>[!error] 청크사이즈가 다름에도 처리속도가 똑같은 이유?

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

>[!error] 반복 TEST 중.. 커넥션풀 Time Out 문제??

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
