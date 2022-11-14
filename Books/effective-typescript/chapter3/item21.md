# 아이템21. 타입 넓히기

타입스크립트가 작성된 코드를 체크하는 정적 분석시점에, 변수는 가능한 값들의 집합 타입을 가진다.

- 정적분석: 소스코드의 실행없이 정적으로 프로그램의 문제를 찾는 과정

상수를 이용해서 변수를 초기화 할 때, 타입을 명시하지 않으면 타입 체커는 **지정된 값을 가지고 할당 가능한 값들의 집합을 유추하여 타입을 결정**한다. 타입스크립트에서는 이러한 과정을 **‘넓히기(widening)’** 이라고 한다.

```tsx
interface Vector3 {
  x: number;
  y: number;
  z: number;
}
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}

let x = 'x';
let vec = {
  x: 10,
  y: 20,
  z: 30,
}
getComponent(vec, x); // ERROR
// TS2345: Argument of type 'string' is not assignable to parameter of type '"x" | "y" | "z"'.
```

위의 함수는 런타임에서는 문제 없지만 IDE 에서는 오류가 발생한다. `getComponent` 함수는 두 번째 매개변수(`axis`) 를 `‘x’ | ‘y’ | ‘z’` 로 예상했지만 x 타입은 할당시점에 **‘넓히기(widening)’** 가 동작해서 string 타입으로 추론되었다. 즉, `string` 타입은 `‘x’ | ‘y’ | ‘z’` 타입에 할당하지 못하니 오류가 발생한 것이다.

```tsx
const a: string = 'a';
const word: 'a' | 'b' | 'c' = a; // TS2322: Type 'string' is not assignable to type '"a" | "b" | "c"'.
```

타입 넓히기가 진행될 때, 주어진 값에 대한 추론 가능한 값들이 여러 개이기 때문에 상당히 애매하다.

```tsx
const mixed = ['x', 1];
```

mixed 변수의 타입은 어떻게 유추될 수 있을까?

- `(’x’ | 1)[]`
- `[’x’ | 1]`
- `[string | number]`
- `readonly [string | number]`
- `(string | number)[]`
- `readonly (string | number)[]`
- `[any, any]`
- `any[]`

주어진 정보가 충분하지 않다면 어떤 타입으로 추론되어야 하는지 알 수가 없다. 따라서 타입스크립트는 작성자의 의도를 추측하게 된다. 하지만 타입스크립트가 똑똑하다고해도 항상 추측한 답이 작성자의 의도와 일치할 수는 없다.

처음예제에서 타입스크립트는 아래와 같은 예제를 예상했기 때문에 `x` 의 타입을 `string` 으로 추론했다.

```tsx
let x = 'x';
x = 'a';
x = 'Four score and seven years ago...';
```

타입스크립트는 `x` 의 타입을 `string` 으로 추론할 때, 명확성과 유연성 사이의 균형을 유지하려고 한다.

타입스크립트는 **넓히기 과정을 제어**할 수 있도록 몇 가지 방법을 제공한다.

### **1. const**

`let` 대신 `const` 를 사용하면 **더 좁은 타입**이 된다.

```tsx
const x = 'x'; // const x: "x"
let x = 'x'; // let x: string
```

`const` 는 재할당이 불가능하므로 타입스크립트는 의심의 여지없이 더 좁은 타입인 `“x”` 로 추론한다.

그러나 `const` 객체나 배열을 다룰 경우 여전히 문제가 있다.

```tsx
const mixed = ['x', 1];
```

배열의 경우 튜플 타입을 추론해야 할지, 요소들을 어떤 타입으로 추론해야 할지 알 수 없다.

```tsx
const v = {
  x: 1,
}
v.x = 3; // O
v.x = '3'; // X
v.y = 4; // X
v.name = 'Pythagoras'; // X
```

객체의 경우 타입스크립트 넓히기 알고리즘은 **각 요소를 `let` 으로 할당**된 것처럼 다룬다. 그래서 `v` 의 타입은 `{ x: number }` 가 된다. `v.x` 에 `number` 타입의 다른 숫자를 재할당하는 것은 가능하지만, `string` 타입은 안된다. 또한 다른 속성을 추가하지도 못한다. 따라서 객체는 한번에 만드는 것이 좋다.

타입 추론의 강도를 직접 제어하기 위해서는 타입스크립트의 기본동작을 재정의 해야 한다.

```tsx
const v: { x: 1|3|5 } = {
  x: 1, // x: 1 | 3 | 5
}
```

### **2. 타입체커에 추가적인 문맥을 제공**

예를 들면 함수의 매개변수로 값을 전달하는 것과 같다.

### **3. const 단언문을 사용**

“const 단언문”은 변수 선언에 사용하는 `let`, `const` 와는 다른 것이다. “const 단언문”은 공간에 대한 기법이다.

```tsx
const v1 = {
  x: 1,
  y: 2,
}
// { x: number;  y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
}
// { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const
// { readonly x: 1; readonly y: 2; }
```

값 뒤에 `as const` 를 사용하면 타입스크립트는 최대한 좁은 타입으로 추론한다.

배열을 튜플 타입으로 추론할 때도 `as const` 를 사용할 수 있다.

```tsx
const a1 = [1, 2, 3]; // number[]
const a2 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

타입 넓히기로 인해 오류가 발생하고 있다면 명시적 타입 구문이나 const 단언문을 추가하는 것을 고려해야 한다.
