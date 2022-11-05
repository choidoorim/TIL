# @Autowired 필드 명, @Qualifier, @Primary
조회 빈이 2개 이상일 때 문제를 해결하는 여러 방법이 있다.

1. `@Autowired` 필드/파라미터명 매칭
2. `@Qualifier` -> `@Qualifier` 끼리 매칭 ->  빈 이름 매칭
3. `@Primary` 사용

## @Autowired 필드명 매칭
<기존 코드>
```java
@Autowired
private final DiscountPolicy discountPolicy;
```

<필드명을 빈 이름으로 변경>
일반적으로 필드명을 보고, 생성자의 경우 파라미터명을 보고 여러 개의 빈(`fixDiscountPolicy, rateDiscountPolicy`) 중에서 똑같은 이름의 빈(`rateDiscountPolicy`)을 가지고 있는 것을 가져와 주입한다.
```java
@Autowired
private final DiscountPolicy rateDiscountPolicy;
```


```java
@Component  
public class OrderServiceImpl implements OrderService {  
    private final MemberRepository memberRepository;  
    private final DiscountPolicy discountPolicy;  

    @Autowired  
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {  
        this.memberRepository = memberRepository;  
        this.discountPolicy = rateDiscountPolicy;  
    }
```

타입에 매칭되는 빈이 한 개 있을 때는 문제가 되지 않지만, 여러 개가 존재할 경우 필드명을 매칭하는 시나리오가 추가되는 것이다.
1. 타입 매칭
2. 타입 매칭 결과가 2개 이상일 때 필드 명, 파라미터명으로 빈 이름 매칭

## @Qualifier 사용
`@Qualifier` 는 추가구문자로 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다.


```java
@Component  
@Qualifier("mainDiscountPolicy")  
public class RateDiscountPolicy implements DiscountPolicy {

@Component  
@Qualifier("fixDiscountPolicy")  
public class FixDiscountPolicy implements DiscountPolicy {
```

<생성자 자동 주입 예시>
```java
@Autowired  
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {  
    this.memberRepository = memberRepository;  
    this.discountPolicy = discountPolicy;  
}
```

`@Qualifier` 를 추가구문자로 지정해주면, 주입시에 `@Qualifier` 를 파라미터에 붙여주면 등록한 이름의 빈을 찾아서 주입해준다.

만약 `@Qualifier` 를 사용해서 주입할 때 해당 이름(`mainDiscountPolicy`)을 찾지 못하면 어떻게 될까? 그러면 `mainDiscountPolicy` 라는 **이름의 스프링 빈을 추가로 찾는다**. 하지만 `@Qualifier` 는 `@Qualifier` 를 찾는 용도로만 사용하는게 명확하고 좋다.

1. `@Qualifier` 끼리 매칭을 한다.
2. 이름을 찾지 못한다면 빈 이름을 매칭
3. `NoSuchBeanDefinitionException` 예외 발생

## @Primary 사용
`@Primary` 는 우선순위를 정하는 방법으로 편하긴 하지만 한계점이 존재한다. `@Autowired` 시에 여러 빈이 매칭되면, `@Primary` 가 우선권을 가진다.
```java
@Component  
@Primary  
public class RateDiscountPolicy implements DiscountPolicy {

@Component  
public class FixDiscountPolicy implements DiscountPolicy {
```
`@Primary` 만 붙여주면 된다. `@Qualifier` 의 단점은 주입 받을 때 모든 코드에 `@Qualifier` 를 붙여주어야 한다는 것이다. 반면에 `@Primary` 를 사용하면 `@Qualifier` 를 붙일 필요가 없다.

**"`@Primary`, `@Qualifier` 활용 사례"**
코드에서 자주 사용하는 메인 데이터베이스의 커넥션을 획득하는 스프링 빈이 있고, 코드에서 특별한 기능으로 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있다고 가정해보자.      
메인 데이터베이스의 커넥션을 획득하는 스프링 빈은 @Primary 를 적용해서 조회하는 곳에서 @Qualifier 지정 없이 편리하게 조회하고, 서브 데이터베이스 커넥션 빈을 획득할 때는 @Qualifier 를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게 유지할 수 있다.
물론 이때 메인 데이터베이스의 스프링 빈을 등록할 때 @Qualifier 를 지정해주는 것은 상관없다.

### 우선순위
`@Primary` 는 기본값 처럼 동작하는 것이고, `@Qualifier` 는 매우 상세하게 동작한다.       
스프링은 넓은 범위의 선택권인 자동보다는 좁은 범위의 선택권인 수동이 우선 순위가 높다.    
따라서 `@Qualifier` 가 우선권이 높다.
