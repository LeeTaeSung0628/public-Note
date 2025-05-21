# Spring MVC의 Service와 ServiceImpl 구조

- 해당 구조가 갖는 장점이 무엇인가?
먼저 Serviced에 인터페이스를 구현하여 세부 구현체를 숨기고 인터페이스를 바라보게 함으로써 클래스간의 의존관계를 줄이는것 이다.
좀 더 쉽게 정리하면,
	하나의 인터페이스를 구현하는 여러개의 구현체가 있고, 기능에 따라 적절한 구현체가 드어감으로써 다형성을 주기위함이다.

- 하지만, 인터페이스 하나에 구현체 한개만 사용하는경우는 어떠한가?
이렇게 된다면, 의존관계를 줄여주는 효과도,  다형성을 주는 효과도 없게된다.
	하지만 보통의 경우 한개의 기능을하는 인터페이스를 여러기능의 구현체로 나누는 일은 쉽게 일어나지 않는다.

``` java
public interface CardPaymentService {
    void pay();
}

public class ShinhanCardPaymentService implements CardPaymentService{
    private ShinhanCard shinhanCard;

    @Override
    public void pay() {
        shinhanCard.pay(); //신한 카드 결제 API 호출
        // 결제를 위한 비즈니스 로직 실행....
    }
}
```
### 하나의 인터페이스의 하나의 구현체
- 위와 같은 경우, 하나의 인터페이스에 하나의 구현체를 갖지만, 향후 추가적으로 구현체가 더 생길여지가 있으니, 인터페이스를 두는것이 바람직 하다고 할 수 있다.

그렇다면 향후 구현체가 추가될 계획이 없는 기능들 까지 인터페이스를 만들어주어야 하는가?
- 그렇지 않다, 예를들어 간단하게 아이디를 기반으로한 조회기능 등은 인터페이스를 구현하지 않고 바로 서비스 객체를 생성하는것이 옳다.
``` java
public interface ChangePasswordService {
    public void change(MemberId id, PasswordDto.ChangeRequest dto);
}

public class ByAuthChangePasswordService implements ChangePasswordService {
    private MemberFindService memberFindService;

    @Override
    public void change(MemberId id, PasswordDto.ChangeRequest dto) {
        if (dto.getAuthCode().equals("인증 코드가 적합한지 로직 추가...")) {
            final Member member = memberFindService.findById(id);
            final String newPassword = dto.getNewPassword().getValue();
            member.changePassword(newPassword);
            // 필요로직...
        }
    }
}

public class ByPasswordChangePasswordService implements ChangePasswordService {
    private MemberFindService memberFindService;

    @Override
    public void change(MemberId id, PasswordDto.ChangeRequest dto) {
        if (dto.getPassword().equals("비밀번호가 일치하는지 판단 로직...")) {
            final Member member = memberFindService.findById(id);
            final String newPassword = dto.getNewPassword().getValue();
            member.changePassword(newPassword);
        }
    }
}
```
이렇게 비밀번호를 변경하는 기능같은 경우에는 2가지 이상의 경우가 있기때문에 인터페이스로 구현하는것이 옳아보인다.

##### 결론 : 하나의 클래스에 너무 많은 역할을 부여하지 말자. (책임을 몰리지 말자)

---