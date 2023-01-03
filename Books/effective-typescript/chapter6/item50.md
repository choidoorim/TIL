# 아이템50. 오버로딩 타입보다는 조건부 타입을 사용하기

```tsx
function double(x: number | string): number | string;
function double(x: any) {
  return x + x;
}
```

double 함수에 number 타입을 넣으면 number 타입을 반환해야 하고, string 타입을 넣으면 string 타입을 반환해야 한다.

```tsx
const num = double(12); // type - string | number
const str = double('x'); // type - string | number
```

하지만 number 타입을 넣고 string 타입을 반환하는 경우도 포함되어 있다.

제네릭 타입을 이용한다면 타입을 구체적으로 만들 수 있다.

```tsx
function double<T extends number|string>(x: T): T;
function double(x: any) {
  return x + x;
}

const num = double(12); // type - 12
const str = double('x'); // type - "x"
```

타입을 구체적으로 만들었지만 너무 과해졌다. string 타입을 매개변수로 전달하면 string 타입이 반환되야 하지만 `‘x’` 를 매개변수로 전달한다고해서 `‘x’` 가 반환되야 하는 것은 아니다. 실제로 `double` 함수에 `‘x’` 를 넣으면 `‘xx’` 가 되야 한다.

여러 가지 타입 선언으로 분리할 수도 있다. 타입스크립트에서 구현체는 하나지만 타입 선언은 제한없이 만들 수 있다(타입스크립트 코드는 런타임에 반영되지 않기 때문에).

```tsx
function double(x: number): number;
function double(x: string): string;
function double(x: any) {
  return x + x;
}

const num = double(12); // type - number
const str = double('x'); // type - string
```

함수 타입이 명확해졌지만 유니온 타입을 매개변수로 전달할 경우 타입 관련 문제가 발생한다.

```tsx
function f(x: number | string) {
  return double(x); 
  // ~ Argument of type 'string | number' is not assignable to parameter of type 'number'.
  // ~ Argument of type 'string | number' is not assignable to parameter of type 'string'.
}
```

위 `f()` 함수는 `double()` 함수의 반환 타입이 `string | number`  일 것을 기대했을 것이다.

타입스크립트는 오버로딩 타입중에서 일치하는 타입을 찾을 때까지 순차적으로 검색한다. 오버로딩 타입의 마지막 선언(`string`) 까지 검색했을 때, `string | number` 타입은 `string` 타입에 할당할 수 없기 때문에 오류가 발생하게 되는 것이다.

오버로딩 타입인 string | number 타입을 추가해서 해결할 수 있지만 가장 좋은 해결책은 **조건부 타입(Conditional type)**을 사용하는 것이다.

```tsx
function double<T extends number | string>(x: T): T extends string ? string: number;
function double(x: any) {
  return x + x;
}

const num = double(12); // type - number
const str = double('x'); // type - string

function f(x: number | string) { // type - string | number
  return double(x); 
}
```

제네릭 타입을 사용한 방법과 유사하지만 더욱 더 정교하다. 조건부 타입은 자바스크립트의 삼항 연산자와 똑같이 사용하면 된다.

- T 타입이 string 의 부분집합(string or 문자열 리터널, 문자열 리터널의 유니온)인 경우 string 타입을 반환하고, 아닐 경우 number 타입을 반환한다.

만약 오버로딩 타입을 작성중이라면 조건부 타입을 작성해서 개선할 수 있는 부분이 있는지 검토하는 것이 중요하다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입 지원이 가능합니다.
