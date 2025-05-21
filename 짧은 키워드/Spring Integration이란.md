# Spring Integration이란?
- 애플리케이션 내부-외부 사이의 메시징을 가능하게 하는 프레임워크이다.
- Spring Framwork에서 매세징이란 메타데이터와 함께 결합되어있는 이련의 자바 오브젝트를 위한 포괄적인 Wrapper를 말한다. 메시지는 여러개의 헤더로 구성된다.

- Enterprise Integration Patterns은 엔터프라이즈 환경에서 사용하고 있는 다양한 분야(ex. 결제,메일, 각 부서별 서비스) 의 애플리케이션을 통합, 즉 유기적으로 연결해서 효율적으로 적절하게 통합하는 방법을 여러 패턴을 통해 제시했다.
- 하나의 동작을 하는 서비스의 각각의 기능들(프로젝트)이 내/외부 모듈과 접촉하는 부분을 쉽게 구성할 수 있도록 하는 기능들을 제공한다.

### Spring Integration을 구성하는 컴포넌트 종류
- 채널: 한 요소로부터 다른 요소로 메시지를 전달
- 필터: 조건에 맞는 메시지가 플로우를 통과하게 해줌
- 변환기: 메시지 값을 변경하거나 메시지 페이로드의 타입을 다른 타입으로 변환
- 라우터: 여러 채널 중 하나로 메시지를 전달하며 대개 메시지 헤더를 기반으로 함
- 분배기: 들어오는 메시지를 두 개 이상의 메시지로 분할하며, 분할된 각 메시지는 다른 채널로 전송
- 집적기: 분배기와 상반된 것으로 별개의 채널로부터 전달되는 다수의 메시지를 하나의메시지로 결합함
- 서비스 액티베이터: 메시지를 처리하도록 자바 메서드에 메시지를 넘겨준 후 메서드의 반환값을 출력 채널로 전송
- 채널 어댑터: 외부 시스템에 채널을 연결함. 외부 시스템으로부터 입력을 받거나 쓸 수 있음
- 게이트웨이: 인터페이스를 통해 통합플로우로 데이터를 전달
## 3가지의 메인 Component
- Spring Integration은 'pipe and filters' 모델을 구현하기 위해 3가지 핵심 개념으로 구성되어 있다.
####  1. Message 
	header, payload로 구성되어 있는 내용을 포함하고 있는 generic wrapper. 
	컴포넌트 간에 이동되는 실제 데이터이다.

![[Pasted image 20240605160336.png]]

#### 2. Message Channel
	pipes-and-filters 모델의 pipe에 해당.
	컴포넌트간의 메세지 중간 통로 역할을 함으로써 컴포넌트간 디컬플링을 유지 할 수 있도록 하며 interception, monitering 포인트가 될 수 있다.

	다른 주요 기능 중 하나는, 메세지 버퍼 역할을 할 수 있는 Queue로써 동작할 수 있다.
	FIFO방식으로 컨슈머가 가져갈 때 까지 큐에 저장된다.
ex)
@Bean
public MessageChannel sampleChannel() {
	return new DirectChannel();
}
-> DirectChannel은 Point to Point로, 하나의 MessageHeader에게 Message를 전달한다.
이외에도 다양한 체널종류가 있다.


#### 3. Message Endpoint
	pipes-and-filters 모델의 filters에 해당.
	Spring integration상에서 채널을 통해서 메세지를 받고, 소비하는 주체이며 하나의 클래스이다.
	여기서 말하는 EndPoint란 Spring integration이 구성할 파이프라인의 끝단이 아닌, 파이프라인 중간에서 메세지를 변경하거나 필터링 하거나, 다른 채널로 라우팅하는 요소이다.
``` java
ex)
@MessageEndpoint
public class serverEndpoint {
	...
}
```

- Spring integration에서 일련의 작업들을 정의한 플로우를 integration flow 라고 하는데, 이 플로우가 Message Endpoint로 구성되어있다.

- 엔드포인트는 작업 타입에 따라 크게 그 종류를 나눌 수 있는데 Transformer(변형), Filter(필터링), Router(메세지를 특정 채널로 전송), Splitter(메세지를 분리하여 여러 채널로 전송), Aggregator(splitter의 반대), Service Activator(메세지로 특정 작업을 수행할 수 있는 핸들러를 붙일 수 있는 엔드포인트), Channel Adapter(외부 시스템과 입출력이 가능)가 있다.
## Filter 란?
- 통합 파이프라인 중간에 위치하며, 조건을 기반으로 플로우의 전 단계로부터 다음단계로의 메세지 전달에 조건을 달 수 있다.
ex)
``` java
@Filter( inputChannel="numberChannel", outputchannel="evenNumberChannel" )
public boolean evenNumberFilter( Integer number ) {
	return number % 2 == 0;      //숫자를 받아 짝수만 전달
}
```
## ServiceActivator 란?
- 입력체널로 부터 메세지를 수신하고, 이 메세지를 MessageHandler 인터페이스를 통해 구현한 클래스에 전달(서비스호출)한다.
- 서비스를 메시징 시스템에 연결하기 위한 앤드포인트이다.
- 입력 채널이 설정되어 있어야 하고, 서비스가 값을 리턴하도록 구현했다면 출력 채널도 설정해야한다.
- **만약 출력체널을 설정하지 않았을 때 메세지에 "return address"가 있다면 이 헤더에 지정한 체널로 응답을 전송한다.
- MessageChennel메서드를 입력해서 파이프라인을 구축한다,
``` java
ex)
@ServiceActivator(inputChannel = "sampleChannel") { }
```

---