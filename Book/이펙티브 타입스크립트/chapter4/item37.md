# 아이템37. 공식 명칭에는 상표를 붙이기

타입스크립트의 구조적 타이핑(Structural Type System) 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.

```tsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({ x: 3, y: 4 });
const vec3D = {
  x: 3,
  y: 4,
  z: 1
};
calculateNorm(vec3D);
```

위 코드는 구조적 타입 시스템에서는 문제가 없지만 사실 `Vector2D` 를 사용해야 하는 것이 맞다. 만약 calculateNorm 함수가 3차원 벡터를 허용하지 않게 하기 위해서는 `Nominal-Typing`(공식명칭) 을 사용하면 된다.

`Nominal-Typing` 개념을 타입스크립트에서 흉내내서 사용하기 위해서는 상표(brand) 를 붙이면 된다.

- `Nominal-Typing` : C++, Java, Swift 와 같은 언어에서 사용하는 타입 검사 방법이다. 같은 이름에 같은 자료형으로 선언된 경우에만 호환된다. ([참고: nominal vs structural typing](https://flow.org/en/docs/lang/nominal-structural/))

```tsx
interface Vector2D {
  __brand: '2d'
  x: number;
  y: number;
}
function vec2D(x: number, y: number): Vector2D {
  return {x, y, __brand: '2d'};
	// or return {x, y} as Vector2D;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4));
const vec3D = {
  x: 3,
  y: 4,
  z: 1
};
calculateNorm(vec3D);
// Argument of type '{ x: number; y: number; z: number; }' is not assignable to parameter of type 'Vector2D'.
// Property '__brand' is missing in type '{ x: number; y: number; z: number; }' but required in type 'Vector2D'.ts(2345)
```

```tsx
type Brand<T, K> = T & { __brand: K };
type USD = Brand<number, 'USD'>;
type EUR = Brand<number, 'EUR'>;

let usd = 10 as USD;
let eur = 20 as EUR;
usd = eur; // Error
```

상표(`__brand`)를 사용해서 `calculateNorm` 함수가 `Vector2D` 타입만 받는 것을 보장한다. 그러나 악의적으로 `vec3D` 변수 값에 `__brand: ‘2d’` 를 사용하는 것을 막을 수는 없다. 단순한 실수를 방지하기는 좋다.

이런 상표 기법은 **타입 시스템에서도 동작하지만, 런타임에서 상표를 검사하는 것과 동일한 효과**를 얻을 수 있다. 타입시스템이기 때문에 런타임 오버헤드를 없앨수 있고, 추가 속성을 붙일 수 없는 string 이나 number 같은 내장 타입도 상표화 할 수 있다.

예를 들어 절대 경로를 사용해 파일 시스템에 접근하는 함수를 가정해보자. 런타임에서는 절대 경로로 시작하는지 체크하기 쉽지만, 타입 시스템에서는 절대 경로를 판단하기 어렵기 때문에 상표기법을 사용한다.

```tsx
type AbsolutePath = string & { __brand: 'abs' };
function listAbsolutePath(path: AbsolutePath) {
  //...
}
// isAbsolutePath 함수의 결과 값이 true 일 경우 매개변수로 전달한 인자의 타입은 AbsolutePath 가 된다.
function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith('/');
}
```

`string` 타입이면서 `__brand` 속성을 가지는 객체를 만들 수는 없다. `AbsolutePath` 는 온전히 타입 시스템 영역에서만 존재하는 것이다.

만약 path 값이 절대 경로와 상대 경로가 모두 될 수 있다면 타입을 정제해주는 타입 가드를 사용해서 오류를 방지할 수 있다.

```tsx
function f(path: string) {
  if (isAbsolutePath(path)) {
    path; // path: AbsolutePath
    listAbsolutePath(path);
  }
  listAbsolutePath(path); // Argument of type 'string' is not assignable to parameter of type 'AbsolutePath'. Type 'string' is not assignable to type '{ __brand: "abs"; }'.ts(2345)
}
```

오류를 방지하기 위해서 `listAbsolutePath(path as AbsolutePath);`  처럼 **타입 단언을 사용해서 오류를 방지할 수 있지만 단언문을 지양**해야 한다. 단언문을 사용하지 않고 `AbsolutePath` 타입을 얻기 위한 유일한 방법은 `AbsolutePath` 타입을 매개변수로 받거나 타입이 맞는지 체크하는 것이다.

이진 검색을 하는 경우를 보자.

```tsx
const binarySearch = <T>(xs: T[], x: T): boolean => {
  let low = 0;
  let high = xs.length;

  while (high >= low) {
    const mid = low + Math.floor(high - low / 2);
    const v = xs[mid];
    if (v === x) {
      return true;
    }
    [low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
  }
  return false;
}
```

이진 검색은 배열이 정렬된 상태에서만 사용이 가능하다. 따라서 배열이 정렬되어 있지 않다면 잘못된 결과가 나온다. 타입스크립트 타입 시스템에서는 목록이 정렬되어 있다는 의도를 표현하기 어렵다.

```tsx
type SortedList<T> = T[] & { __brand: 'sorted' };

function isSorted<T>(xs: T[]): xs is SortedList<T> {
  for (let i = 1; i < xs.length; i++) {
    if (xs[i] < xs[i - 1]) {
      return false;
    }
  }
  return true;
}

function binarySearch<T>(xs: SortedList<T>, x: T): boolean {
  let low = 0;
  let high = xs.length;

  while (high >= low) {
    const mid = low + Math.floor(high - low / 2);
    const v = xs[mid];
    if (v === x) {
      return true;
    }
    [low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
  }
  return false;
}

const arr = [1, 2, 4, 8, 10];
const isSortedArray = isSorted(arr);
if (isSortedArray) {
  arr;
  binarySearch(arr, 4);
}
binarySearch(arr, 4); // Error
```

정렬된 `SortedList<T>` 타입을 사용하기 위해서는 `isSorted` 를 호출해서 정렬되어 있음을 타입 가드를 통해 증명해야 한다. 이와 같은 예제는 타입체커를 유용하게 사용하는 방법 중 하나이다.
