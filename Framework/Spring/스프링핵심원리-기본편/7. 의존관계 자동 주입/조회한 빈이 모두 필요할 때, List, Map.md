# 조회한 빈이 모두 필요할 때, List, Map

```java
public class AllBeanTest {

    @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

        Assertions.assertThat(discountService).isInstanceOf(DiscountService.class);
        Assertions.assertThat(discountPrice).isEqualTo(1000);

        int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");
        Assertions.assertThat(rateDiscountPrice).isEqualTo(2000);
    }

    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policyList;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policyList) {
            this.policyMap = policyMap;
            this.policyList = policyList;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policyList = " + policyList);
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
}
```

```
policyMap = {fixDiscountPolicy=hello.core.discount.FixDiscountPolicy@1838ccb8, rateDiscountPolicy=hello.core.discount.RateDiscountPolicy@6c2ed0cd}
policyList = [hello.core.discount.FixDiscountPolicy@1838ccb8, hello.core.discount.RateDiscountPolicy@6c2ed0cd]
```

DiscountService는 모든 DiscountPolicy 를 `policyMap` 이 변수에 주입받아 저장한다.     
`discount()` 메서드는 discountCode 인자로 `fixDiscountPolicy` 가 넘어올 경우 map 에서 `fixDiscountPolicy` 스프링 빈을 찾아서 실행한다. 당연히 `rateDiscountPolicy` 가 넘어오면 `rateDiscountPolicy` 스프링 빈을 찾아서 실행해준다.

이렇게 다형성을 통해서 유연한 전략 패턴을 사용할 수 있다.
