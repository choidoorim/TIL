# 아이템34. 부정확한 타입보다는 미완성 타입을 사용하기

일반적으로 타입이 구체적일 수록 버그를 더 많이 잡고 타입스트립트에서 제공하는 많은 기능들을 더 사용할 수 있다. 타입을 구체적으로 작성할 때는 정밀도에 많은 신경을 써야 한다. 타입이 실수 하기 쉽고, 잘못된 타입은 오히려 없는 것보다 못할 수 있기 때문이다.

각각 다른 형태의 좌표배열을 가지는 몇가지 타입이 있다.

```tsx
interface Point {
  type: 'Point';
  coordinates: number[];
}
interface LineString {
  type: 'Point';
  coordinates: number[];
}
interface Polygon {
  type: 'Point';
  coordinates: number[];
}
type Geometry = Point | LineString | Polygon;
```

큰 문제는 없지만 좌표에서 사용되는 `number[]` 타입이 추상적이다. `coordinates` 속성은 위도와 경도를 나타내기 때문에 튜플 타입으로 선언하는 것이 좋다.

```tsx
type GeoPosition = [number, number];
interface Point {
  type: 'Point';
  coordinates: GeoPosition;
}
```

더 나은 코드인 것 같지만 그렇지 않다. 왜냐하면 GeoPosition 은 위도와 경도만을 생각하고 만들었지만 고도 등 다른 정보들이 있을 수 있기 때문이다. 결과적으로 타입을 세밀하게 만들려고 했지만 너무 과하고, 오히려 부정확해졌다.

JSON 으로 정의된 Lisp(함수형 언어) 와 비슷한 언어의 타입 선언을 작성한다고 해보자. 입력 값의 종류는 아래와 같다.

1. 모두 허용
2. 문자열, 숫자 배열 허용
3. 문자열, 숫자, 알려진 값으로 시작하는 배열 허용
4. 각 배열이 받는 값의 개수가 정확한지 확인
5. 각 배열이 받는 값의 타입이 정확한지 확인

```tsx
type Expression1 = any;
type Expression2 = number | string | any[];

const tests: Expression2[] = [
  10,
  'red',
  true, // TS2322: Type 'boolean' is not assignable to type 'Expression2'.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue'],
  ['**', 2, 31], // '**' 은 함수가 아니므로 오류가 발생해야 한다
  ['rgb', 255, 128, 64],
  ['rgb', 255, 0, 127, 0],
]
```

정밀도를 높이기 위해 튜플의 첫 번째 요소에 문자열 리터널 타입의 유니온을 사용해보자.

```tsx
type FnName = '+' | '-' | '*' | '>' | '<' | 'case' | 'rgb';
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;

const tests: Expression3[] = [
  10,
  'red',
  true, // TS2322: Type 'boolean' is not assignable to type 'Expression3'.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue'],
  ['**', 2, 31], // Type '"**"' is not assignable to type 'FnName'.
  ['rgb', 255, 128, 64],
  ['rgb', 255, 0, 127, 0],
]
```

정밀도를 유지하면서 전에 발생하지 않았던 오류를 잡아냈다.

타입스크립트에서 함수의 매개변수의 개수를 알기 위해서는 최소한 하나의 인터페이스를 추가해야 한다. 여러 인터페이스를 호출 표현식으로 한 번에 묶을 수는 없기 때문에, 각 인터페이스를 나열해서 호출 표현식을 작성한다.

```tsx
interface MathCall {
  0: '+' | '-' | '/' | '*' | '>' | '<';
  1: Expression4;
  2: Expression4;
  length: 3;
}
interface CaseCall {
  0: 'case';
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4 | 6 | 8 | 10 | 12 | 14 | 16;
}
interface RGBCall {
  0: 'rgb';
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4;
}

type CallExpression = MathCall | CaseCall | RGBCall;
type Expression4 = number | string | CallExpression;

const tests: Expression4[] = [
  10,
  'red',
  true, // TS2322: Type 'boolean' is not assignable to type 'Expression4'.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue'],
  ['**', 2, 31], // TS2322: Type '"**"' is not assignable to type '"rgb"'.
  ['rgb', 255, 128, 64],
  ['rgb', 255, 0, 127, 0, 1], // ~  is not assignable to type 'Expression4'. ~  is not assignable to type 'RGBCall'.
]
```

이제 무효한 표현식에는 전부 오류가 발생하게 된다. 타입 정보는 더 정밀해졌지만 결과적으로 이전보다 개선되었다고 보기는 어렵다. 왜냐면 **잘못된 코드에서 오류를 잡아내긴하지만 오류메시지는 더 난해**해졌다. 새 타입 선언은 더 구체적이지만 자동 완성을 방해한다.

또한 타입 작성의 **복잡성으로 인해 버그**가 발생할 가능성도 높아졌다. 예를 들면 Expression4 는 모든 수학연산에 2 개의 매개변수가 필요하지만 `+, *` 연산자가 더 많은 매개변수를 받을 수도 있다. 입력을 `-` 로 바꿔주는 `-` 연산자는 1 개의 매개변수만 필요로 한다.

```tsx
const tests: Expression4[] = [
  ['-', 12], // TS2322: Type '"-"' is not assignable to type '"rgb"'.
  ['+', 1, 2, 3], // TS2322: Type '"-"' is not assignable to type '"rgb"'.
  ['*', 2, 3, 4], // TS2322: Type '"-"' is not assignable to type '"rgb"'.
]
```

코드를 더 정밀하게 만들려는 시도는 너무 과했고, 그로인해 코드가 더 부정확해졌다.

타입이 구체적으로 정의된다고해서 정확도가 무조건 올라가지는 않는다. **타입에 의존하기 시작하면 부정확함으로 인해 발생하는 문제**가 더 커지게 된다.

즉, **부정확한 타입은 타입이 없는 것보다 못하다**.
