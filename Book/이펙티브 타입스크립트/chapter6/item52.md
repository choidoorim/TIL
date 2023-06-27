# 아이템52. 테스팅 타입의 함정에 주의하기

프로젝트를 공개하기 위해서는 테스트 코드를 작성하는 것은 필수이고, 타입 선언도 테스트를 거쳐야 한다. 그러나 타입 선언을 테스트하기는 매우 어렵다. 따라서 타입스크립트의 단언문을 통해 쉽게 해결하는 법이 있지만, 이 방법에는 몇 가지 문제가 있다.

타입 선언 파일을 테스팅할 때는 단순히 함수를 실행만 하는 방식을 일반적으로 적용하게 되는데, 그 이유는 라이브러리 구현체의 기존 테스트 코드를 복사하면 간단히 만들 수 있기 때문이다.

square 라는 함수의 런타임 동작을 테스트한다면 아래와 같은 테스트코드를 작성할 수 있다.

```tsx
test('square a number', () => {
  square(1);
  square(2);
});
```

이 테스트 코드는 square 함수의 실행에서 오류가 발생하지 않는지만 체크한다. 그런데 반환 값에 대해서는 체크하지 않기 때문에 실제로는 실행 결과에 대한 테스트는 하지 않은게 된다. 그렇기에 함수의 반환 타입을 체크하는 테스트 코드를 짜는 것이 훨씬 좋은 테스트이다.

반환 값을 특정 타입의 변수에 할당하여 간단히 반환 타입을 체크할 수 있는 방법을 알아보자.

첫 번째, 불필요한 변수를 만들어야 한다. 반환 값을 할당하는 변수는 샘플코드처럼 쓰일 수도 있지만, 일부 린팅 규칙을 비활성해야 한다.

```tsx
function assertType<T>(x: T) {}

assertType<number[]>(['john', 'paul'].map(name => name.length));
```

불필요한 변수 문제를 해결하지만, 다른 문제점이 있다.

두 번째, 두 타입이 동일한지 체크하는 것이 아니라 할당 가능성을 체크하고 있다.

```tsx
const n = 12;
assertType<number>(n);
```

그러나 객체의 타입을 계산할 때 문제를 발견할 수 있을 것이다.

```tsx
const beatles = ['john', 'pau1', 'george', 'ringo'];

function assertType<T>(x: T) {}
assertType<{ name: string }[]>(
  beatles.map((name) => ({ name, inYelloSubmarine: name === 'ringo' })),
);
```

현재 map 함수를 사용해서 `{ name: string, inYelloSubmarine: boolean }` 타입을 반환하고 있다. 하지만 할당 가능성 체크는 통과한다. `inYelloSubmarine` 속성에 대한 부분이 체크되지 않은 것이다.

assertType 에 함수를 넣어서 체크를 해보면 이상한 결과가 나온다.

```tsx
const add = (a: number, b: number) => a + b;
assertType<(a: number, b: number) => number>(add); // 정상

const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); // 정상
assertType<(a: number, b: string) => number>(double); // 정상
assertType<(a: string, b: number) => number>(double); // 에러 - Type 'string' is not assignable to type 'number'.
```

`double` 함수가 할당 가능검사에 통과할 수 있는 이유는 **타입스크립트의 함수는 더 적은 매개변수를 가진 함수에 할당이 가능**하기 때문이다.

```tsx
const g: (x: string) => any = () => 12; // 정상
```

선언된 것보다 적은 매개변수를 가진 함수를 할당하는 것은 아무런 문제가 없다.

그렇다면 제대로 된 assertType 의 사용법은 무엇일까? `Parameters` 과 `ReturnType` 제네릭 타입을 통해 함수의 **매개변수 타입과 반환 타입만 분리하여 테스트**할 수 있다.

```tsx
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!;
assertType<[number, number]>(p); // 에러 - TS2345: Argument of type '[x: number]' is not assignable to parameter of type '[number, number]'.   Source has 1 element(s) but target requires 2.
assertType<[number]>(p); // 정상

let r: ReturnType<typeof double> = null!;
assertType<number>(r); // 정상
```

타입 선언을 체크하는 것은 어렵지만 반드시 해야하는 작업이다. 문제점을 방지하기 위해 `dtslint` 같은 도구를 사용하는 것도 선택지중 하나이다.
