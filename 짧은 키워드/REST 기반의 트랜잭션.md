# REST 기반의 트랜잭션

### 모놀리틱 아키텍처에서는 일반적으로 데이터베이스의 트렌젝션에 의존한다.
![[Pasted image 20240603173517.png]]

하지만, MSA의 경우 각 서비스마다 다른 데이터베이스를 사용하는 것이 일반적이고, 이를 하나의 데이터베이스 트렌젝션으로 처리하는 것은 기술적으로 어렵고, 처리한다 해도 긴 트렌젝션이 발생하기 때문에 효용도 적다.
![[Pasted image 20240603180042.png]]
## TCC ( Try-Confirm/Cancel)
- tcc는 분산된 REST 시스템들 간의 트랜젝션을 HTTP와 REST원칙으로 접근하여 해결하는 방법이다.


REST API 호출은 한 번에 끝내는 것이 아니라, 2번의 걸쳐서(Try / Confirm) 진행하게 된다.
트렌젝션의 All-or-Nothing을 TCC의 REST API를 호출을 시도(Try)하고 전부 확정(Confirm)하거나 전부 취소(Cancel)하는 것으로 구현된다.

- Spring RestTemplate을 사용하여 HTTP 요청(POST)을 보냈을 때, try 요청의 경우 정상적인 HTTP응답(HttpStatus.CREATED)를 받으면 HTTP BODY에는 JSON형태로 Confirm하게 하거나 Cancel 할 수 있는 URL이 담겨 있다.

### @RestController를 사용하여 HTTP POST Method와 연결할 수 있다.
- 여기서 Service에서 반환받은 값을 기준으로 Confirm 할지 Cancel할지 선택하게 된다.
- 중요한 것은 여기서 실제로 데이터베이스 테이블에 변경이 있는것 이 아닌, Confirm되었을때 그때 처리가 된다.

### 이후 @PutMapping을 사용하여 HTTP PUT Method와 연결된다. 
- 여기서 받은 반환값을 토대로 Service에서는 resource 필드(JSON)을 역질렬화 하고 이를 사용하여 그때 실제로 데이터베이스에 있는 테이블을 변경하게 된다.

## 예약한 리소스 문제
- Try는 리소스를 사용하기 전에 예약하는 것이다. 만약 4.구매 주문 생성에서 Try만 하고, 실패했다면 REST로 통신은 기다리고 있던(Try만 한 상태) 두 API에는 Confirm이 전달되지 않아 예약만 된 상태로 남아있게 된다.
- 예약된 상태는 특정 리소스를 점유하고 있다는 의미이며, 리소스를 점유하고 있는 동안에는 다른 API에서 해당 리소스를 사용하는 것은 제한된다.
- 따라서, 4. 행위에서 Try만 하고 실패했다면, 예약한 리소스까지 해제해주어야 한다.

- 분산된 환경에서 리소스를 해제하는 것은 쉬운 문제가 아닌데, TCC매커니즘에서는 Cancel과 Timeout 두가지 방법으로 예약된 리소스를 해제한다.
![[Pasted image 20240604102140.png]]

REST커뮤니케이션 관전에서 자세하게 설명하면,
1. TCC REST API Consumer(여기선 OrderService)가 Try요청
2. TCC REST API Provider (여기선 StockService/PaymentService)는 응답으로 Confirm하거나 Cancel할 수 있는 URI를 반환
3. 이를 사용하여 API Consumer는 DELETE HTTP Method로 예약한 리소스에 대한 해제를 요청한다.


## 엄격한 일관성과 결과적 일관성
![[Pasted image 20240604144026.png]]

- 클라이언트가 주문을하고, OrderService는 StockService와 PaymentService로 Try한다.
 그리고 구매 주문을 생성 후 Confirm하였다. StockService는 재고 처리에 성공을 한 반면,  PaymentService는 결제에 실패한다.
 이경우에는 어떻게 일관성을 유지할 수 있을까?

#### 엄격한 일관성
- 관계형 데이터베이스에서 트랜젝션을 처리할 때에는 데이터 적합성을 보장해야 하기 때문에 엄격한 일관성 모델을 사용한다.
- 하지만 결제시스템 하나의 문제로 모든 비즈니스가 멈추게 되는 문제가 발생한다.(보통은 멈추는게 맞다)

#### 결과적 일관성
- StockService와 PaymentService는 OrderService로부터 받은 Confirm요청을 Queue나 Log파일에 큐잉 하고, 이를 비동기적으로 처리한다. Confirm처리 과정에서 오류가 나는 경우 계속해서 재시도하여 결국(언젠가) 처리하게 한다.
- 이렇게 단기적으로 일관성을 잃더라도(클라이언트 입장에서는 성공했다고 느끼지만, 실제 결제처리가 되지 않았을 수도 있다.) 결국에서는 일관성을 유지하는 모델을 결과적 일관성 이라고 한다.
- 단, 결과적 일관성 모델은 단기적으로 일관성을 잃어버렸을 때를 대비한 화면 처리 등이 필요하다.

- ex) 아마존에서 전자책을 구입한 후, 결제 과정이 진행되었고 이후 카드가 정상처리되지 않는 메일을 받아, 2일후에 제대로 결제처리를 하였다.
