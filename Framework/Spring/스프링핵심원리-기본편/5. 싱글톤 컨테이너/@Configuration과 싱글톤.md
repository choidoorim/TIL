## @Configuration과 싱글톤
```java
@Configuration  
public class AppConfig {  
    @Bean()  
    public MemberRepository memberRepository() {  
        return new MemoryMemberRepository();  
    }  
    @Bean  
    public MemberService memberService() {  
        return new MemberServiceImpl(memberRepository());  
    }  
  
    @Bean  
    public DiscountPolicy discountPolicy() {  
//        return new FixDiscountPolicy();  
        return new RateDiscountPolicy();  
    }  
    @Bean  
    public OrderService orderService() {  
        return new OrderServiceImpl(memberRepository(), discountPolicy());  
    }  
}
```

위의 Configuration 에서 현재 OrderService 를 호출할 때와 MemberService 를 호출할 때 각각 MemoryMemberRepository 를 호출한다. 즉 MemoryMemberRepository 셍성이 new 를 통해 2 번 되는 것이므로 싱글톤이 깨지는 것이 아닌가라는 생각이 들 수 있다.

```
@Bean memberService -> new MemoryMemberRepository()  
@Bean orderService -> new MemoryMemberRepository()
```

```java
public class OrderServiceImpl implements OrderService {  
    private final MemberRepository memberRepository;  
    
    //...
  
    public MemberRepository getMemberRepository() {  
        return memberRepository;  
    }  
}
```

```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;  
    
    //...
  
    public MemberRepository getMemberRepository() {  
        return memberRepository;  
    }  
}
```

실제로 OrderServiceImpl 와 MemberServiceImpl 에서 가져오는 memberRepository 는 서로 다른 인스턴스여야 한다.

```java
@Test  
void configurationTest() {  
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);  
  
    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);  
    OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);  
  
    MemberRepository memberRepository1 = memberService.getMemberRepository();  
    MemberRepository memberRepository2 = orderService.getMemberRepository();  
  
    System.out.println("memberService memberRepository1 = " + memberRepository1);  
    System.out.println("orderService memberRepository2 = " + memberRepository2);  
}
```

<img width="674" alt="스크린샷 2022-09-29 오후 10 14 14" src="https://user-images.githubusercontent.com/63203480/193041248-f8b0cbbb-2beb-4ed7-9c70-b35fb5d63030.png">

하지만 실제로 확인해보면 결과는 같은 값을 가진 인스턴스인 것을 볼 수 있다.

## @Configuration과 바이트코드 조작의 마법
스프링 컨테이너는 싱글톤 레지스트리(저장소)이다. 따라서 스프링 빈이 어떻게든 싱글톤이 되도록 보장해줘야 한다. 하지만 스프링이 자바 코드까지 조작하는 것은 어렵기 때문에 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.

- `new AnnotationConfigApplicationContext(AppConfig.class)` 을 통해 넘기는 Class 도 스프링 빈으로 등록이 된다.

```java
@Test  
void configurationDeep() {  
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);  
    AppConfig bean = ac.getBean(AppConfig.class);  
  
    System.out.println("bean = " + bean.getClass());  
}
```

<img width="530" alt="스크린샷 2022-09-29 오후 10 29 56" src="https://user-images.githubusercontent.com/63203480/193044785-1bf39f0c-f470-4d6a-abc1-3835b795bb5f.png">

실제로 AppConfig 를 조회해보면 `class hello.core.AppConfig` 까지 조회되어야 하는 것이 `$$EnhancerBySpringCGLIB$$9db1a227` 라는 이상한 값이 추가로 붙어있는 것을 확인할 수 있다.
만약 순수 클래스였다면 `class hello.core.AppConfig` 까지만 조회되었어야 했다.

- 바이트코드: JVM 이 이 이해할 수 있는 언어로 변환된 자바 코드

이것은 내가 만든 클래스가 아닌 스프링이 `CGLIB` 라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속답은 임의의 다른 클래스를 만들고, 다른 클래스를 스프링 빈으로 등록한 것이다.      
임의의 다른 클래스가 바로 싱글톤이 보장되도록 해주는 것이다. 아마 아래와 같이 바이트 코드를 조작해서 작성되어 있을 거라고 예상할 수 있다.

```java
public MemberRepository memberRepository() {
	if (memberRepository 가 이미 스프링 컨테이너에 등록되어 있으면) {
		return 스프링 컨테이너에서 찾아서 반환
	} else { // 스프링 컨테이너에 등록되어 있지 않으면
		기존 로직을 호출해서 MemoryMemberRepository 를 생성하고 스프링 컨테이너에 등록
		return 반환
	}
}
```

`AppConfig bean = ac.getBean(AppConfig.class);` 을 통해 인스턴스를 조회할 수 있는 이유는 `AppConfig@CGLIB`  라는 인스턴스가 AppConfig 라는 이름을 사용하고 있기 때문인 것이다.

그렇다면 `@Configuration` 을 적용하지 않고 `@Bean` 만 적용한다면 어떻게 될까?

```java
//@Configuration  
public class AppConfig {
```

<img width="284" alt="스크린샷 2022-09-29 오후 10 44 56" src="https://user-images.githubusercontent.com/63203480/193048590-1a12c63a-5cef-4292-a630-0f1784679231.png">

위와 같이 `CGLIB` 기술이 적용되지 않은 순수 클래스가 나오는 것을 확인할 수 있다.

<img width="257" alt="스크린샷 2022-09-29 오후 10 47 13" src="https://user-images.githubusercontent.com/63203480/193049063-d5360fee-71cb-4b98-ae0f-b495434e8437.png">

하지만 memberRepository 가 3 번이나 출력이 된다. 즉, 싱글톤이 깨진 것이다.

`@Bean` 만 사용해도 스프링 빈으로 등록은 되지만, 싱글톤을 보장하지는 않는다.
스프링 설정 정보는 항상 `@Configuration` 을 사용하면 된다.
