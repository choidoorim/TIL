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
