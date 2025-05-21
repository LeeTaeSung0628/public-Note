# Equals() 와 HashCode() 재정의
``` java
@Test
@DisplayName("같은 객체를 equals 비교")
void equals() {
    //given
    Menu friedChicken = new Menu("후라이드치킨", 16_000);
    Menu friedChicken2 = new Menu("후라이드치킨", 16_000);
    //when & then
    assertThat(friedChicken).isEqualTo(friedChicken2);
}
```

헤당 코드와 같이 구현한다면, false를 출력한다.
이유는 객체의 equals메서드는 주소값이 서로 다른 객체는 다른객체로 판단하기 때문이다.
#### String은 equals 잘쓰잖아? => String은 String pool에서 중복생성을 막고, 같은 값을 생성한다면 재사용하기 때문~

## 재정의 해서, 객체가 아닌 내부 값을 비교하는 equals를 만들었다.
이때, 왜 HashCode도 재정의 해야하나?

### HashCode의 규약에는 다음과 같은 사항이 존재한다.

#### * equals(object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode값은 항상 같아야한다

해당 규약으로 인하여, 서로다른 객체의 해쉬값을 통일시켜주어야 한다.

