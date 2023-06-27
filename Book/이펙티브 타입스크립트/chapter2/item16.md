# 아이템 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트의 객체 모델에는 이상한 점들이 있으며, 이 중 일부는 타입스크립트 타입 시스템으로 모델링 되기 때문에 **자바스크립트 객체 모델을 이해하는 것이 중요**하다.

자바스크립트에서 **객체란 키와 값 쌍의 모음**이다. 키는 보통 문자열이며 값은 어떤 것이든 될 수 있다. 파이썬이나 자바에서 볼 수 있는 해시가능 객체라는 표현이 자바스크립트에서는 없다. 만약에 복잡한 객체를 키로 사용하려고 하면, `toString` 메서드가 호출되어 객체가 문자열로 반환된다.

```jsx
let x = {};
x[[1, 2, 3]] = 2
console.log(x); // { '1,2,3': 2 }
```

특히 키값을 숫자는 사용할 수 없다. 만약 **숫자를 사용하려고하면 자바스크립트 런타임에서 문자열로 변환할 것**이다.

자바스크립트에서 **배열은 분명하게 객체**이다.

```jsx
typeof []; // object
```

그렇기에 숫자 인덱스를 사용하는 것은 당연한 것이다.

```jsx
const foo = {
  0: 1,
}
const bar = [1];

console.log(foo[0]); // 1
console.log(bar[0]); // 1
```

이상하지만 앞의 인덱스들은 문자열로 변환되어 사용된다. 그렇기 때문에 문자열 키를 사용해도 값을 찾을 수 있다.

```jsx
console.log(foo['0']); // 1
console.log(bar['0']); // 1
```

`Object.keys` 를 사용해서 나열해보면 키가 문자열로 출력된다.

```jsx
const foo = [1, 2, 3];
console.log(Object.keys(foo)); // ['0', '1', '2']
```

타입스크립트는 이런 혼란스러운 점을 바로 잡기 위해서 숫자키를 허용하고 문자열 키와 다른 것으로 인식한다. 런타임에서는 문자열 키로 인식하기 때문에 타입 체크 시점에 오류를 잡을 수 있어 유용하다.

```tsx
function get<T>(array: T, k: string) {
  return array[k]; //TS7053: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{}'. No index signature with a parameter of type 'string' was found on type '{}'.
}
```

하지만 타입스크립트에서 `Object.keys` 는 문자열로 반환된다.

```tsx
const xs = [1, 2, 3];
const keys = Object.keys(xs);
for (const key in xs) {
  console.log(typeof key); // type -> string
  const x = xs[key]; // type -> number
}
```

이처럼 `string` 타입이 `number` 타입에 할당되어 동작되는 것이 이상하게 보일 수 있다. 이것은 타입스크립트에서의 **배열을 순회하는 코드 스타일에 대한 실용적인 허용**이다. 하지만 이런 순회방법은 좋지 않다.

인덱스에 신경쓸 필요가 없다면 `for-of,` 인덱스의 타입이 중요하다면 `Array.prototype.forEach` 를 사용하면 된다. 만약 루프를 중간에 멈춰야 한다면 C 언어 스타일인 `for(;;)` 를 사용하면 된다.

```tsx
const xs = [1, 2, 3];
for (const x of xs) {
  console.log(typeof x); // number, number, number
}

xs.forEach((value, index) => {
  console.log(typeof value, index); // number 0, number 1, number 2
})

for (let i = 0; i < xs.length ; i++) {
  const x = xs[i];
  if (x < 0) break;
}
```

타입이 불확실할 경우 대부분 `for-in` 보다 `for-of`, `for(;;)` 가 더 빠르다.

배열의 인덱스 시그니처가 `number` 라면 입력한 값은 `number` 타입이지만 실제로 런타임에서는 `string` 타입이다. 일반적으로는 `string` 대신 `number` 타입의 인덱스 시그니처를 사용할 일이 없다. 만약 숫자를 사용해서 인덱스 항목을 지정하기 보다는 Array 또는 튜플 타입을 대신 사용하는 것이 좋다.

만약 어떤 길이를 가지는 **배열과 비슷한 형태의 튜플을 사용하고 싶다면 타입스크립트에 있는 `ArrayLike` 타입을 사용**하면 된다. ArrayLike 타입은 **Array.prototype 에 있는 함수들이 필요하지 않을 때 사용**하면 된다. ArrayLike 를 사용하더라도 키 값은 여전히 문자열이라는 것을 잊지 말아야 한다.

```tsx
const tupleLike: ArrayLike<string> = {
  '0': 'A',
  '1': 'B',
  length: 2,
} // 정상
```

![Untitled 1](https://user-images.githubusercontent.com/63203480/200846155-4327dad7-b7aa-4705-bbee-1cf5a3706760.png)
![Untitled](https://user-images.githubusercontent.com/63203480/200846211-0fc0bff0-7ec1-4d36-a114-b842d0689c04.png)
