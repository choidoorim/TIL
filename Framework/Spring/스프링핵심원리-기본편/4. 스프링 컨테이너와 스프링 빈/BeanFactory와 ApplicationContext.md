#### BeanFactory
- 스프링 컨테이너의 최상위 인터페이스이다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다. 예를 들면  `getBean()` 과 같은 기능을 제공해주는 것이다.
- 지금까지 사용한 기능들은 대부분 BeanFactory 에서 제공해주는 것이다.

#### ApplicationContext
- ApplicationContext 은 BeanFactory 의 기능을 모두 상속받아서 사용한다.
- 어플리케이션을 개발할 때는 빈을 관리하고 조회하는 것 제외하고, 수 많은 부가기능이 필요하다. 따라서 BeanFactory 만 상속받는 것이 아닌 아래와 같이 다른 인터페이스들도 상속받아서 사용하고 있다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,  
      MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```

- MessageSource - 국제화 기능제공
    - 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력하는 웹사이트가 대표적이다.
- EnvironmentCapable - 환경변수
    - 로컬, 개발, 운영 등을 구분해서 처리한다.
- ApplicationEventPublisher - 어플리케이션 이벤트
    - 이벤트를 발행하고 구독하는 모델을 편리하게 지원한다.
- ResourcePatternResolver - 편리한 리소스 조회
    - 파일, 클래스 경로, 외부 URL 등에서 리소스를 편리하게 조회할 수 있도록 한다.

#### 정리
- ApplicationContext 는 BeanFactory 의 기능을 상속받는다.
- ApplicationContext 는 빈 관리기능 + 편리한 부가 기능을 제공한다.
- BeanFactory 를 직접사용할 일은 거의 없다. 부가기능이 포함된 ApplicationContext 를 사용한다.
- BeanFactory 나 ApplicationContext 를 스프링 컨테이너라고 한다.
