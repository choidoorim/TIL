# 아이템54. 객체를 순회하는 노하우

```tsx
const obj = {
  one: 'uno',
  two: 'dos',
  three: 'tres',
};

for (const k in obj) {
  // TS7053: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ one: string; two: string; three: string; }'.
  // No index signature with a parameter of type 'string' was found on type '{ one: string; two: string; three: string; }'.
  const v = obj[k];
}
```

위 예제는 정상적으로 실행되지만, IDE 에서는 오류가 발생한다. 오류는 `obj` 객체를 순회하는 `k` 상수와 관련된 오류이다. 상수 k 의 타입은 `const k: string` 이지만 obj 객체에는 `one, two, three` 3 개의 키 값만 존재한다.  즉, `k` 와 `obj` 키의 타입이 서로 다르게 추론돼서 오류가 발생한 것이다. `k` 의 타입을 더욱 구체적으로 명시해준다면 오류는 사라지게 된다.

```tsx
let k: keyof typeof obj;
for (k in obj) {
  const v = obj[k];
}
```

그렇다면 `k` 의 타입이 `one, two, three` 가 아닌 `string` 으로 추론되는 이유는 무엇일까? 아래 코드도 위의 예제와 동일한 오류가 발생한다.

```tsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  // TS7053: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'ABC'.   No index signature with a parameter of type 'string' was found on type 'ABC'.
  for (const k in abc) {
    const b = abc[k];
  }
}
```

그렇기에 *`let* k: *keyof* ABC;` 을 추가해서 해결할 수 있다. 오류의 내용이 잘못되었다고 생각할 수 있지만 타입스크립트는 정확히 오류를 표시한 것이다. 아래 코드는 정상적으로 동작한다.

```tsx
const x = {
  a: 'a',
  b: 'b',
  c: 2,
  d: new Date(),
}
foo(x);
```

이처럼 `ABC` 타입에 할당 가능한 객체는 `a, b, c` 이외에 **다른 속성이 존재할 수 있기 때문**에(타입스크립트는 구조적 타이핑 시스템이기 때문), 타입스크립트는 `ABC` 타입의 키를 `string` 으로 선택할 수 밖에 없는 것이다.

만약 타입 문제 없이 단지 객체와 키 값을 순회하고 싶다면 `Object.entries` 를 사용하면 된다.

```tsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k; // a, b, c, d - type: string
    v; // A, B, 2, Wed Jan... - type: any
  }
}

const x = {
  a: 'A',
  b: 'B',
  c: 2,
  d: new Date(),
}
foo(x);
```

객체를 다룰 때는 항상 프로토타입의 오염 가능성을 염두해야 한다. for-in 구문을 사용하면 객체의 정의에 없는 속성이 등장할 수도 있다.

```tsx
Object.prototype.z = 3;
const obj = { x: 1, y: 2 };
for (const k in obj) {
  console.log(k); // x, y, z
}
```

실제 작업에서는 `Object.prototype` 에 루프가 가능한 속성을 절대 추가하면 안된다.

객체를 순회하면서 key 와 value 를 얻기 위해서는 `let k: keyof T` 와 같은 keyof 선언이나 `Object.entries` 를 사용하면 된다.
