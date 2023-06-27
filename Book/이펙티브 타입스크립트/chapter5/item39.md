# 아이템39. any 를 구체적으로 변형해서 사용하기

`any` 는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다. `any` 타입에는 모든 숫자, 문자열, 배열, 객체, 정규식, 함수, 클래스, DOM 엘리먼트 뿐만 아니라 `null` 과 `undefined` 까지도 포함한다. 즉, 일반적으로 any 보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 있기 때문에 더 구체적인 타입을 찾아 안정성을 높이도록 해야 한다.

예를 들면 any 타입을 정규식이나 함수에 넣는 것은 권장되지 않는다.

```tsx
function getLengthBad(array: any) {
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

만약 `any` 를 사용한다면 `getLengthBad` 함수보다는 `getLength` 함수가 더 좋다.

1. 함수 내의 `array.length` 타입이 체크가 된다.
2. 함수의 반환타입이 `any` 가 아닌 `number` 로 추론된다.
3. 함수 호출시 인자 값이 배열인지 체크한다.

```tsx
getLengthBad(123);
getLength(123); // Argument of type 'number' is not assignable to parameter of type 'any[]'.ts(2345)
```

만약 함수의 매개변수가 객체이지만 값을 알 수 없다면 `{[key: string]: any}` 처럼 선언하면 된다.

```tsx
const obj: {[key: string]: any} = {
  a: 1,
  b: 'string',
  c: {
    foo: 'foo'
  }
};
```

함수의 매개변수가 객체지만 값을 알 수 없다면 `{[key: string]: any}` 대신 모든 비기본형 타입을 포함하는object 타입을 사용할 수 있다. `object` 타입은 객체의 키들을 확인(열거)할 수 있지만, 속성에 접근할 수 없다는 점에서 `{[key: string]: any}` 와는 다르다.

```tsx
const obj: object = {
  a: 1,
  b: 'string',
  c: {
    foo: 'foo'
  }
};
console.log(obj); // { a: 1, b: 'string, c: { foo: 'foo' } }
console.log(obj.a); // Property 'a' does not exist on type 'object'.ts(2339)
```

객체지만 속성에 접근할 수 없어야 한다면 unknown 타입이 필요한 상황일 수도 있다.

함수의 타입에도 단순히 any 를 사용하면 안된다. 최소한으로 구체화할 수 있는 방법이 있다.

```tsx
// 1. 매개변수 없이 호출하는 경우
type Fn0 = () => any;
// 2. 매개변수 1개 
type Fn1 = (arg: any) => any;
// 3. 모든 개수의 매개변수, Typescript Function 타입과 동일
type FnN = (...args: any[]) => any;
```

이런 함수 타입들이 단순 `any` 타입보다는 구체적이다. 그리고 `...args: any` 으로 선언해도 되지만 `...args: any[]` 으로 선언한 것은 `any` 보다는 `any[]` 이 배열인 것을 알 수 있어서 구체적이기 때문이다.
