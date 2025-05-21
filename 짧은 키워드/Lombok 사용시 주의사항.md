# Lombok 사용시 주의사항

## 1. @AllArgsConstructor / @RequiredArgsConstructor 사용시
	- 위 두개의 어노테이션을 편리하게 생성자를 자동으로 생성해 주지만, 주의를 요할 필요가 있다.

- 어떠한 클래스에서 순서대로 인자를 받는 생성자가 있다고 했을 때,  개발자가 임의로 순서를 변경할 경우, 리펙토링은 전혀 작동하지 않고, lombok이 개발자가 인지하지 못하는 사이에 순서에 맞춰 두 필드를 변경해 버린다.
- 그렇기 때문에 순서의 구애받지 않는 @Builder 어노테이션을 사용한는 것을 권장하고 있다.

## 2. 무분별한 @EqualsAndHashCode 사용
	 Mutable(변경가능한)객체에 아무런 파라미터 없이 그냥 사용하는 경우에 문제가 발생할 수 있다.

- 동일한 객체임에도 불구하고 Set으로 필드값을 변경하게 되면, hashCode가 변경되면서 찾을 수 없게되는 부분이 있다. 

## 3. @Data 사용금지
	- 위에서 설명한 @RequiredArgsConstructor 및 @EqualsAndHashCode를 포함하고 있기 때문에 사용을 피하는 것이 좋다.

## 4. @Value 사용 금지
	- 불변 클래스를 생성해주는 @Value또한, @EqualsAndHashCode와 @AllArgsConstructor를 포함하고 있기 때문에 사용을 피하는것이 좋다. 
	- 불변클래스 이기 때문에 @EqualsAndHashCode는 문제가 되지 않지만, AllArgsConstructor가 문제를 일으킬 가능성이 있다.


---