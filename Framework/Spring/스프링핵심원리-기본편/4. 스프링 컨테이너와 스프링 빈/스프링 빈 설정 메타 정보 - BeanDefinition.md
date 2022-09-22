스프링이 다양한 설정 형식(Java, XML 등) 을 지원할 수 있는 것은 `BeanDefinition` 이라는 추상화가 있기 때문이다.

즉, 역할과 구현을 개념적으로 잘 나눴기에 가능한 것이다. XML 을 읽어서 `BeanDefinition` 을 만들면되고, 자바 코드를 읽어서 `BeanDefinition` 을 만들면 된다. 그렇게 된다면 스프링 컨테이너는 자바 코드인지, XML 인지 몰라도 되고 오로지 `BeanDefinition` 만 알면 된다.

`BeanDefinition` 을 빈 설정 메타정보라고 하고, `@Bean` , `<bean>` 당 각각 하나씩 메타 정보가 생성된다. 그리고 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.

추상화에만 의존하기 때문에 가능한 것이다.
<img width="825" alt="스크린샷 2022-09-22 오후 11 02 30" src="https://user-images.githubusercontent.com/63203480/191768128-02ea02ff-b004-43cb-8e67-5d9d72cdf599.png">


`AnnotationConfigApplicationContext`  는  `AnnotatedBeanDefinitionReader` 를 사용해서 `AppConfig.class`  를 읽고  `BeanDefinition` 을 생성한다. 아래 사진은`AnnotationConfigApplicationContext` 클래스 내부이다.

<img width="949" alt="스크린샷 2022-09-22 오후 11 09 25" src="https://user-images.githubusercontent.com/63203480/191769707-ed6d8bff-7a64-47f2-85b1-6c8f2cda6786.png">

새로운 형식의 설정 정보가 추가된다면, XxxBeanDefinitionReader 를 만들어서 `BeanDefinition` 을 생성하면 된다.

## BeanDefinition 살펴보기
```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);  
  
@Test  
@DisplayName("빈 설정 메타정보 확인")  
void findApplicationBean() {  
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();  
    for (String beanDefinitionName: beanDefinitionNames) {  
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);  
  
        if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {  
            System.out.println("beanDefinitionName = " + beanDefinitionName + " beanDefinition " + beanDefinition);  
        }  
    }  
}
```

bean 을 조회해서 확인해보면 다양한 `BeanDefinition` 정보가 있는 것을 확인할 수 있다.

<img width="928" alt="스크린샷 2022-09-22 오후 11 24 34" src="https://user-images.githubusercontent.com/63203480/191773509-803a0b84-a21f-4a32-8f67-ddcfc83f9e7a.png">
- BeanClassName: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음) 
- factoryBeanName: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
- factoryMethodName: 빈을 생성할 팩토리 메서드 지정, 예) memberService  
- Scope: 싱글톤(기본값)
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연처리 하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명 
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

`BeanDefinition` 을 직접 생성해서 스프링 컨테이너에 등록할 수도 있지만 실무에서 `BeanDefinition` 를 직접 정의하거나 사용할 일은 거의 없다.
따라서 `BeanDefinition` 에 대해서 깊이 있게 이해보다는 스프링이 다양한 형태의 설정 정보를 `BeanDefinition` 으로 추상화해서 사용하는 것 정도만 이해하면 된다.
