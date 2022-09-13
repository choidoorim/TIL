## 제어의 역전(Inversion of Control)
- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 개발자 입장에서는 자연스러운 흐름이다.
- 반면에 AppConfig 가 등장하면서 구현 객체는 자신의 로직을 실행하는 역할만 담당하게 된다. 프로그램의 제어 흐름은 이제 AppConfig 가 가져가게 된다.
- 프로그램에 대한 모든 제어 흐름은 AppConfig 가 가지고 있다. 심지에 "xxxImpl" 객체의 생성도 AppConfig 가 담당한다.
- 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.

## 프레임워크 vs 라이브러리
- 프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그것은 프레임워크가 맞다
- 예를 들면 JUnit 이 Test, BeforEach 등의 Annotation 을 통해 내가 구현한 로직을 실행하고 제어권을 JUnit 이라는 프레임워크가 대신 가져가고 자신의 라이프사이클을 통해 실행한다.
- 반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 그것은 라이브러리이다.

## 의존관계 주입(DI)
- OrderServiceImpl 은 DiscountPolicy 인터페이스에 의존한다. 실제 어떤 구현 객체가 사용될지는 모른다.
```java
public class OrderServiceImpl implements OrderService {  
    private final DiscountPolicy discountPolicy;
```
- 의존관계를 정적인 클래스 의존관계와 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계 둘을 분리해서 생각해야 한다.

### 1. 정적인 클래스 의존관계
클래스가 사용하는 Import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 정적인 의존관계는 어플리케이션을 실행하지 않아도 분석할 수 있다.
인텔리제이에서 폴더-> 우클릭 -> Diagrams 를 통해 의존관계를 확인할 수 있다.

### 2. 동적인 클래스 의존관계
- 어플리케이션 실행시점(런타임)에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 **의존관계 주입** 이라고 한다.
- 객체 인스턴스를 생성하고, 그 참조 값을 전달해서 연결한다.
```java
// AppConfig
public class AppConfig {
	private DiscountPolicy discountPolicy() {  
	//  return new FixDiscountPolicy();  
	    return new RateDiscountPolicy();  
	}
	public OrderService orderService() {  
	    return new OrderServiceImpl(discountPolicy());  
	}
}

public class OrderServiceImpl implements OrderService {  
    private final DiscountPolicy discountPolicy;
    
    public OrderServiceImpl(DiscountPolicy discountPolicy) {  
	    this.discountPolicy = discountPolicy;  
    }
    //...
}
// discountPolicy에는 객체 인스턴스의 참조 값이 연결되어 있다.
// 실제로는 RateDiscountPolicy 객체의 인스턴스 참조 값이 연결되어 있다.
```
- 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
- 의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다. 정적인 클래스 의존관계를 변경하지 않는다는 것은 어플리케이션 코드를 변경하지 않는다는 것이다.

## IoC 컨테이너, DI 컨테이너
- 기존에 개발한 **AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해주는 것**을 IoC 컨테이너, **DI 컨테이너**라고 한다. 즉 AppConfig 는 우리가 직접 만들어 관리했지만, 이제는 스프링의 DI 컨테이너에서 대신 관리해주는 것이다.
- 어플리케이션 구성에 대한 조립을 하기 때문에 **어셈블러** 라고 하기도하고, 객체를 생성하고 주입하기에 **오브젝트 팩토리** 등으로 불리기도 한다.


