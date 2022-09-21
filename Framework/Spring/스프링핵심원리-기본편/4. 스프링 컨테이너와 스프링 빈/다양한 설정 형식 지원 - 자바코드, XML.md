- 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다. ex) 자바 코드, XML, Groovy 등

<img width="884" alt="스크린샷 2022-09-21 오후 10 46 52" src="https://user-images.githubusercontent.com/63203480/191521143-36370d83-69ca-4ca1-bd9c-16bc072e24e2.png">

- XML 이라는 문서정보를 설정정보로 사용하거나 임의로 직접구현해서 만들 수도 있다. 최근에는 자바 코드기반의 AnnotationConfig 를 많이 사용하고, 과거에는 XML 을 많이 사용했다. (!스프링을 공부하면 XML 정도는 알아두는 것이 좋다)
- 최근에는 스프링 부트를 사용하면서 XML 기반의 설정은 잘 사용하지 않는다. 레거시 프로젝트들이 XML 기반으로 되어 있고, XML 을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수도 있는 장점도 존재한다.

- 자바 코드가 아닌 것들은 `/src/main/resources` 경로에 두면 된다.
- 아래의 두 개의 코드는 같은 뜻이다.

```xml
<bean id="memberService" class="hello.core.member.MemberServiceImpl" >  
    <constructor-arg name="memberRepository" ref="memberRepository"/>  
</bean>

<bean id="memberRepository" class="hello.core.member.MemoryMemberRepository"/>
<!--
constructor-arg 태그가 생성자로 넘긴다는 뜻이다.
-->
```

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
    //...
}
```

- xml 기반의 `appConfig.xml` 스프링 설정 정보와 자바 코드로 된 `AppConfig.java` 설정 정보를 비교해보면 거의 비슷하다는 것을 알 수 있다.

스프링은 이처럼 매우 유연하게 설정들을 다루고 적용할 수 있다.