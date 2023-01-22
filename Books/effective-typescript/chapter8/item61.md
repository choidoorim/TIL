# 아이템61. 의존성 관계에 따라 모듈 단위로 전환하기

점진적 마이그레이션을 할 때는 모듈 단위로 각개격파하는 것이 이상적이다. 한 모듈을 골라서 타입 정보를 추가하면, 해당 모듈이 의존(임포트)하는 모듈에서 비롯되는 타입 오류가 발생하게 된다. 의존성과 관련된 오류 없이 작업하기 위해서는 **다른 모듈에 의존하지 않는 최하단 모듈부터 작업을 시작해서 최상단에 있는 모듈을 마지막으로 완성**해야 한다.

프로젝트 내부에 존재하는 모듈은 서드파티 라이브러리에 의존하지만 서드파티 라이브러리는 해당 모듈에 의존하지 않기 때문에, 서드파티 라이브러리 타입 정보를 가장 먼저 해결해야 한다. 일반적으로는 `@types/Xxx` 모듈을 설치하면 된다. `ex) npm install --save-dev @types/lodash`

외부 API 를 호출하는 경우도 있기 때문에 외부 API 의 타입 정보도 추가해야 한다.

모듈 단위로 마이그레이션을 시작하기 전에, 모듈 간의 의존성 관계를 시각화하면 많은 도움이 된다. 대부분의 프로젝트에서는 **의존성의 촤하단에 util 종류의 모듈이 위치하는 패턴이 일반적**이다. 오래된 프로젝트일수록 개선이 필요한 부분을 마주치겠지만 타입스크립트 마이그레이션이 목적이기 때문에 리팩터링을 해서는 안된다.

## 선언되지 않은 클래스 멤버

자바스크립트에서는 클래스 멤버 변수를 선언할 필요가 없지만, 타입스크립트에서는 명시적으로 선언해야 한다.

```jsx
// Javascript - 정상
class Greeting {
  constructor(name) {
    this.greeting = 'Hello';
    this.name = name;
  }

  greet() {
    return this.greeting + ' ' + this.name;
  }
}
```

```tsx
// Typescript - 에러
class Greeting {
  constructor(name) {
    this.greeting = 'Hello'; // Property 'greeting' does not exist on type 'Greeting'.
    this.name = name; // Property 'name' does not exist on type 'Greeting'
  }

  greet() {
    return this.greeting + ' ' + this.name;
  }
}
```

따라서 아래와 같이 수정해야 한다. IDE 의 도움을 받아서 속성을 추가할 수 있다.

```tsx
class Greeting {
  greeting: string;
  name: any;
  constructor(name) {
    this.greeting = 'Hello'; // Property 'greeting' does not exist on type 'Greeting'.
    this.name = name; // Property 'name' does not exist on type 'Greeting'
  }

  greet() {
    return this.greeting + ' ' + this.name;
  }
}
```

## 타입이 바뀌는 값

아래 코드는 자바스크립트에서는 문제 없지만 타입스크립트에서는 오류가 발생한다.

```jsx
const state = {}
state.name = 'New York';
state.capital = 'Albany';
```

```tsx
const state = {}
state.name = 'New york'; // Property 'name' does not exist on type '{}'
state.capital = 'Albany'; // Property 'capital' does not exist on type '{}'.
```

위 예제와 같은 경우에는 객체를 한 번에 생성하면 간단히 오류를 해결할 수 있다.

한 번에 생성하긴 어려운 경우에는 **임시 방편으로 타입 단언문**을 사용할 수 있다.

```tsx
interface State {
  name: string;
  capital: string;
}

const state = {} as State;
state.name = 'New York';
state.capital = 'Albany';
```

마이그레이션이 완료되었다면 타입 선언으로 바꿔야 한다.

마지막으로 테스트 코드를 타입스크립트로 전환하면 된다. 로직 코드그 테스트 코드에 의존하지 않기 때문에, 테스트 코드는 항상 의존성 관계도의 최상단에 위치하며 마이그레이션의 마지막 단계가 되는 것은 자연스럽다. 그리고 최하단부터 마이그레이션을 전환해도 테스트 코드는 수행할 수 있었을 것이다. 마이그레이션 중에 테스트를 수행할 수 있다는 것은 엄청난 이점이다.
