# 아이템41. any 의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다. 또한 새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입은 예외적인 경우가 존재한다.

```tsx
function range(start: number, limit: number) {
  const out = []; // Type: any[]
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out; // Type: number[]
}
```

`out` 의 타입은 `any[]` 로 선언되었지만 `number` 타입의 값을 넣는 순간 타입은 `number[]` 가 된다.

타입의 진화는 타입 좁히기와는 다르다. 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화한다.

```tsx

const result = []; // Type: any[]
result.push('a');
result; // Type: string[]
result.push(1);
result; // Type: (string | number)[]
```

또한 조건문의 분기마다 타입이 바뀔 수 있다.

```tsx
let val;
if (Math.random() < 0.5) {
  val = /hello/;
  val; // Type: RegExp
} else {
  val = 12;
  val; // Type: number
}
val; // Type: number | RegExp
```

변수의 초기값이 null 인 경우도 any 의 진화가 일어난다. 보통은 try/catch 블록 안에서 변수를 할당하는 경우에 나타난다.

```tsx
let val = null; // Type: any
try {
  somethingDangerous();
  val = 12;
  val; // Type: number
} catch (e) {
  console.log('alas!');
}
val; // Type: number | null
```

이런 any 타입의 진화는 `tsconfig.json` 에서 `noImplicitAny` 가 설정된 상태에서 변수의 타입이 암시적(Implicit) any 인 경우에만 일어난다. 만약 **명시적으로 타입을 선언한다면 타입이 그대로 유지**된다.

```tsx
let val: any = null; // Type: any
try {
  somethingDangerous();
  val = 12;
  val; // Type: any
} catch (e) {
  console.log('alas!');
}
val; // Type: any
```

암시적 `any` 상태인 변수에 **어떠한 할당도 하지 않고, 사용하려고하면 암시적 `any` 오류가 발생**한다.

```tsx
function range(start: number, limit: number) {
  const out = []; // TS7034: Variable 'out' implicitly has type 'any[]' in some locations where its type cannot be determined.
  if (start === limit) {
    return out; // TS7005: Variable 'out' implicitly has an 'any[]' type.
  }
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

암시적 `any` 타입은 루프를 거쳐도 진화하지 않는다. 아래의 `forEach` 구문은 타입 추론에 영향을 끼치지 않는다.

```tsx
function makeSquares(start: number, limit: number) {
  const out = []; // TS7034: Variable 'out' implicitly has type 'any[]' in some locations where its type cannot be determined.
  range(start, limit).forEach(i => {
    out.push(i);
  });
  return out; // TS7005: Variable 'out' implicitly has an 'any[]' type.
}
```

이런 경우라면 루프를 순회하는 대신 `map` 과 `filter` 메서드를 통해 단일 구문으로 배열을 생성해서 `any` 전체를 진화시키는 방법이 좋다.

```tsx
function makeSquares(start: number, limit: number) {
  const out = [];

  const newOut = range(start, limit).map((i) => i * i);
  return newOut; // Type: number
}
```

타입을 안전하게 지키기 위해서는 암시적 any 를 진화시키는 것보다 명시적 타입구문을 사용하는 것이 더 좋은 설계이다.
