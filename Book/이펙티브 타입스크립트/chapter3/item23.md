# 아이템23. 한꺼번에 객체 생성하기

객체를 생성할 때는 속성을 하나씩 추가하기 보다는 여러 속성을 한 꺼번에 포함해서 생성해야 타입추론에 유리하다.

자바스크립트에서 객체를 생성하는 방식 중 하나는 아래와 같다.

```jsx
const point = {};
point.x = 1;
point.y = 2;
console.log(point); // { x: 1, y: 2 }
```

타입스크립트에서는 위와 같이 객체 속성을 할당한다면 오류가 발생한다.

```tsx
const point = {};
point.x = 1; // TS2339: Property 'x' does not exist on type '{}'.
point.y = 2; // TS2339: Property 'y' does not exist on type '{}'.
```

왜냐하면 point 변수의 타입은 `{}` 으로 추론되기 때문이다.

interface 를 정의한다면 아래와 같이 오류가 발생할 것이다.

```tsx
interface Point {
  x: number;
  y: number;
}

const point: Point = {}; // TS2739 - Type '{}' is missing the following properties from type 'Point': x, y
point.x = 1;
point.y = 2;
```

이러한 문제들은 객체를 한 번에 정의하면 해결할 수 있다.

```tsx
interface Point {
  x: number;
  y: number;
}

const point: Point = {
  x: 1,
  y: 2,
};
```

만약 객체를 나눠서 만들어야 한다면, 타입 단언문을 통해 타입 체커를 통과하게 만들 수 있다.

```tsx
interface Point {
  x: number;
  y: number;
}

const point = {} as Point;
point.x = 1;
point.y = 2;
```

작은 객체 여러 개를 조합해서 큰 객체를 만들어야 할 경우에는 **객체 전개 연산자(`..obj`)**를 통해서 만드는 것이 좋다.

```tsx
const foo = { a: 1 };
const bar = { b: 1 };

const bigObj = { ...foo, ...bar };
console.log(bigObj); // { a: 1, b: 1 }
```

객체 전개 연산자를 사용한다면 타입 걱정 없이 필드 단위로 객체를 생성할 수 있다. 이 때 **새 변수를 선언해서 각각 새로운 타입을 얻을 수 있도록** 하는 것이 중요하다.

```tsx
interface Point {
  x: number;
  y: number;
};

const pt0 = {};
const pt1 = { ...pt0, x: 3 };
const pt: Point = { ...pt1, y: 4 }; // 정상
```

타입에 안전하게 조건부 속성을 추가하고 싶다면 `null` 또는 `{}` 으로 객체 전개를 사용하면 된다.

```tsx
let hasMiddle: boolean = true;
const firstLast = { first: 'Harry', last: 'Truman' };
const president = { ...firstLast, ...(hasMiddle ? { middle: 'S' } : {}) } 
// or: const president = { ...firstLast, ...(hasMiddle && { middle: 'S' }) }
console.log(president); // { first: 'Harry', last: 'Truman', middle: 'S' }
```

편집기에서 추론된 `president` 변수의 타입을 확인하면 아래와 같을 것이다.

```tsx
const president: {
    middle?: string | undefined;
    first: string;
    last: string;
}
```

또한 객체 전개 연산자를 통해 한번에 여러 속성을 추가할 수도 있다.

```tsx
let hasDates: boolean = true;
const nameTitle = { name: 'Durumi', title: 'Typescript' };
const pharaoh = {
  ...nameTitle,
  ...(hasDates ? { start: -2500, end: -2566 } : {}),
};

// type
const pharaoh: {
    start?: number | undefined;
    end?: number | undefined;
    name: string;
    title: string;
}
```

가끔 객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶을 수 있다. 이런 경우 내장형 함수나, 로대시(`Lodash`) 같은 유틸 라이브러리를 사용하는 것이 한 번에 객체를 생성하는 관점에서 올바른 사용법이다.
