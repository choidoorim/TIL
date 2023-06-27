# 아이템53. 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

타입스크립트가 나온 시점에는 자바스크립트가 결함도 많고 개선해야 할 것들이 많은 언어였다. 그리고 클래스, 데코레이터, 모듈 시스템 같은 기능이 없었기에 프레임워크나 트랜스파일러로 보완하는 것이 일반적이였다. 그렇기에 타입스크립트도 초기 버전에는 독립적으로 개발한 클래스, Enum, 모듈 시스템을 포함시켜야 했다.

시간이 지나면서 부족한 부분들을 내장기능으로 추가했다. 그러나 자바스크립트에서 새로 추가된 기능은 타입스크립트 초기 버전때 독립적으로 개발한 것들과 충돌이 발생했다.

따라서 타입스크립트의 초기 버전 형태를 유지하기 위해 자바스크립트 신규 기능을 변형해서 끼워맞추는 방법(1)과 자바스크립트의 신규 기능을 그대로 채택하고 타입스크립트의 초기 버전과 호환성을 포기(2)하는 2 가지의 전략 중 하나를 선택했다.

타입스크립트 진영은 2 번째 전략을 선택했다. 하지만 2번 전략을 세우기 전에 이미 개발되어 사용되고 있던 기능 몇 가지가 있다. 이 기능은 타입 공간(타입스크립트)과 값 공간(자바스크립트)을 혼란스럽게 하기 때문에 사용하지 않는 것이 좋다.

## 열거형(Enum)

값의 모음을 나타내기 위해서 Enum 을 사용한다. 타입스크립트에서도 사용할 수 있다.

```tsx
enum Flavor {
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}

let flavor = Flavor.CHOCOLATE;

Flavor;
console.log(Flavor[0]) // VANILLA
```

값을 나열하는 것보다 실수가 적고 명확하기 때문에 일반적으로 Enum 을 사용하는 것이 좋다. 그러나 타입스크립트의 Enum 에는 몇 가지 문제가 있다.

- 숫자 Enum에 0, 1, 2 외에 다른 숫자가 할당되면 매우 위험하다.
- 상수 Enum 은 보통의 Enum 과 다르게 런타임에서 완전히 제거된다.
- 문자열 Enum 은 런타임의 타입 안전성과 투명성을 제공한다. 그러나 타입스크립트의 구조적 타이핑이 아닌 명목적 타이핑을 사용한다.

타입스크립트의 일반적인 타입들이 할당 가능성을 체크하기 위해서 구조적 타이핑을 사용하지만, 문자열 Enum 은 명목적 타이핑을 사용한다.

```tsx
enum Flavor {
  VANILLA = 'vanilla',
  CHOCOLATE = 'chocolate',
  STRAWBERRY = 'strawberry',
}

let flavor = Flavor.CHOCOLATE;
flavor = 'strawberry'; // TS2322: Type '"strawberry"' is not assignable to type 'Flavor'.
```

- 명목적 타이핑: 타입의 구조가 아닌 타입의 명칭만 가지고 구별되는 방법

Flavor 는 런타임 시점에는 문자열이기 때문에, 자바스크립트에서는 아래와 같이 호출할 수 있다.

```tsx
function scope(flavor: Flavor) { /* ... */ }
```

```jsx
function scope('vanilla'); // Javascript 에서는 정상
```

그러나 타입스크립트에서는 Enum 을 문자열 대신 사용해야 한다.

```jsx
scope('vanilla'); // Error - TS2345: Argument of type '"vanilla"' is not assignable to parameter of type 'Flavor'.
scope(Flavor.VANILLA); // OK
```

이처럼 **자바스크립트와 타입스크립트에서 동작이 다르기 때문에 문자열 열거형은 사용하지 않는 것이 좋다**. 열거형 대신 리터럴 타입의 유니온을 사용하면 된다.

```jsx
type Flavor = 'vanilla' | 'chocolate' | 'strawbeery';

let flavor: Flavor = 'chocolate';
let flavor2: Flavor = 'mint chip'; // TS2322: Type '"mint chip"' is not assignable to type 'Flavor'.
```

리터럴 타입의 유니온은 Enum 만큼이나 안전하며 자바스크립트와 호환되는 장점이 있다. 그리고 IDE 에서 자동완성 기능도 사용할 수 있다.

## 매개변수 속성

클래스를 초기화할 때 속성을 할당하기 위해 생성자의 매개변수를 사용한다.

```tsx
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}
```

타입스크립트는 더 간결한 문법을 제공한다.

```tsx
class Person {
  constructor(public name: string) {}
}
```

위, 아래 `Person` 객체는 동일하게 동작한다. 그러나 매개변수 속성과 관련된 몇 가지 문제점이 존재한다.

```tsx
// 일반 속성
class Person {
  name: string;
  constructor(name: string) {
		this.name = name;
  }
}
// 매개변수 속성
class Person {
  constructor(public name: string) {}
}
```

- 타입스크립트 컴파일은 타입 제거가 이뤄지기 때문에 코드가 줄어들지만, 매개변수 속성은 코드가 늘어나는 문법이다.
- 매개변수 속성이 **런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼** 보인다.
- 매개변수 속성과 일반 속성을 섞어서 사용하면 클래스의 설계가 혼란스러워진다.

아래는 NestJS 에서 실제로 타입스크립트 코드를 컴파일한 뒤의 자바스크립트 코드이다. 매개변수 속성을 사용해도 실제로는 일반 속성으로 컴파일된 것을 확인할 수 있다.

```tsx
export class DiaryController {
  constructor(private readonly diaryService: DiaryService) {}
```

```jsx
let DiaryController = class DiaryController {
    constructor(diaryService) {
        this.diaryService = diaryService;
    }
//...
```

Person 클래스가 아래와 같이 있다.

```tsx
class Person {
  first: string;
  last: string;
  constructor(public name: string) {
    [this.first, this.last] = name.split(' ');
  }
}
```

`first, last, name` 3 가지의 속성이 있지만, `first` 와 `last` 만 속성이 정의되어있고, `name` 은 매개변수 속성에 있어서 일관성이 없다.

클래스에 매개변수 속성만 존재한다면 클래스 대신 인터페이스로 만들고 객체 리터럴을 사용하는 것이 좋다. 구조적 타이핑이라는 특성 때문에 아래 예제처럼 할당할 수 있다는 점을 주의해야 한다.

```tsx
class Person {
  constructor(public name: string) {}
}
const p: Person = {
  name: 'Jed Bartlet',
}
```

매개변수 속성을 사용하는 것은 찬반논란이 있다. 타입스크립트의 다른 패턴들과 이질적이고, 초급자에게는 생소한 문법이라는 것을 기억해야 한다. 또한 매개변수 속성과 일반 속성을 같이 사용하면 설계가 혼란스러워지기 때문에 한 가지만 사용하는 것이 좋다.

## 네임스페이스와 트리플 슬래시 임포트

ECMAScript 2015 이전에는 자바스크립트에 공식적인 모듈 시스템이 없었다. 그래서 각 환경마다 자신만의 방식으로 모듈 시스템을 사용했다. Node.js 는 require 와 module.export 를 사용한 반면, AMD 는 define 함수와 콜백을 사용했다.

타입스크립트도 자체적으로 모듈 시스템을 구축했고, module 키워드와 트리플 슬래시 임포트를 사용했다. ECMAScript 2015 가 공식적으로 모듈 시스템을 도입한 이후, 타입스크립트는 충돌을 피하기 위해서 module 과 같은 기능을 하는 namespace 키워드를 추가했다.

```tsx
namespace foo {
  function bar() {}
}

/// <reference path="other.ts"/>
foo.bar();
```

트리플 슬래시 임포트와 module 키워드는 호환성을 위해 남아 있을 뿐이고, 이제는 ECMAScript 2015 스타일의 모듈(import 와 export)을 사용해야 한다.

## 데코레이터

데코레이터는 클래스, 메서드, 속성에 Annotation 을 붙이거나 기능을 추가하는데 사용할 수 있다.

```tsx
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  @logged
  greet() {
    return 'Hello, ' + this.greeting;
  }
}

function logged(target: any, name: string, descriptor: PropertyDescriptor) {
  const fn = target[name];
  descriptor.value = function () {
    console.log(`Calling ${name}`);
    return fn.apply(this, arguments);
  }
}
```

예를 들어 greet 함수 호출 시 로그를 남기고 싶다면 logged annotation 을 정의할 수 있다.

데코레이터 기능은 처음에 앵귤러 프레임워크를 지원하기 위해 추가되었다. `tsconfig.json` 에 `experimentalDecorators` 속성을 설정하고 사용해야 한다.

하지만 현재까지 표준화가 완료되지 않았기 때문에, 사용 중인 데코레이터가 비표준으로 바뀌거나 호환성이 깨질 가능성이 있다. 앵귤러를 사용하거나, annotation 이 필요한 프레임워크를 사용하는게 아니라면 표준이 되기 전까지 데코레이터는 사용하지 않는 것이 좋다.
