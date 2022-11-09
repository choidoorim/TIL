# 아이템14. 타입 연산과 제너릭 사용으로 반복 줄이기

중복된 코드를 열심히 제거하면서 DRY 원칙을 지켰던 개발자라도 타입에 대해서는 간과했을 수 있다.

타입 중복 문제는 코드 중복 만큼 많은 문제를 발생시킨다. 타입의 중복을 줄일 수 있는 가장 간단한 방법은 이름을 붙이는 것이다.

```tsx
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {
  /* ... */ 
}
```

예를 들어 몇몇 함수가 같은 타입 시그니처를 공유하고 있다고 가정해보자. 그러면 해당 시그니처를 **명명된 타입(Named Type)으로 분리**해 낼 수 있다.

```tsx
function get(url: string, opts: Options): Promise<Response> { /* ... */ }
function post(url: string, opts: Options): Promise<Response> { /* ... */ }

type HTTPFunction = (url: string, opts:Options): Promise<Response>;
const get1: HTTPFunction = (url, opts) => {};
const post1: HTTPFunction = (url, opts) => {};
```

그리고 한 인터페이스가 다른 인터페이스를 **확장**해서 반복을 줄일 수 있다.

```tsx
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
type PersonWithBirthDate = Person & { birth: Date }
```

예를 들어, 전체 어플리케이션의 상태를 표현하는 `State` 타입과 부분을 표현하는 `TopNavState` 가 있는 경우가 있다고 해보자.

```tsx
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

이런 경우에는 TopNavState 를 확장하기 보다는 State 의 부분집합으로 TopNavState 를 정의하는 것이 더 좋은 방법이다.

```tsx
// X
interface State extends TopNavState {
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}

// O
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```

이러한 방법은 전체 어플리케이션 상태를 하나의 인터페이스로 유지할 수 있게 해준다. `State` 를 인덱싱해서 속성의 타입에서 중복을 제거할 수 있다. `State` 의 `pageTitle` 의 타입이 바껴도 `TopNavState` 에 반영이 된다.

**매핑된 타입**을 사용하면 반복되는 코드를 더 줄일 수 있다.

```tsx
type TopNavState = {
  [k in 'userId'| 'pageTitle' | 'recentFiles']: State[k];
}
```

매핑된 타입은 **배열의 필드를 반복문 도는 것과 같은 방식**이다.

```tsx
const numbers = [1, 2, 3];
for (const number in numbers) {
  console.log(number)
}
```

이 패턴은 표준 라이브러리에서도 사용할 수 있으며  `Pick` 이라고 한다. `Pick` 은 제네릭 타입으로 `T, K` 2 가지 타입을 받아서 결과를 return 한다.

```tsx
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

태그된 유니온에서도 다른 형태의 중복이 발생한다.

```tsx
interface SaveAction {
  type: 'save',
}

interface LoadAction {
  type: 'load',
}
type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load'; // 중복
```

Action 유니온을 인덱싱하면 타입의 반복 없이 `ActionType` 을 정의할 수 있다.

```tsx
type ActionType = Action['type'];
```

Action 유니온에 타입을 더 추가하면 ActionType 은 자동적으로 그 타입을 포함하게 된다.

```tsx
//...
interface TestAction {
  type: 'test',
}

type ActionType = Action['type']; // "load" | "save" | "test"
```

보통 update 기능은 create 에서 사용한 매개변수와 동일하며, 타입 대부분이 선택적 필드가 된다.

```tsx
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
```

이런 경우 매핑된 타입과 `keyof` 를 사용하면 `Options` 로부터 `OptionsUpdate` 를 간편하게 만들 수 있다.

```tsx
type OptionsUpdate = {
  [k in keyof Options]?: Options[k];
}
```

이러한 패턴 역시 일반적으로 사용되며 표준 라이브러리에 Partial 라는 기능으로 제공되고 있다.

```tsx
type OptionsUpdate = Partial<Options>;
```

값의 해당하는 형태의 타입을 정의할 수도 있다.

```tsx
const init_options = {
  width: 640,
  height: 400,
  color: '#00FF00',
  label: 'VGA',
}

type Options = typeof init_options;
```

이런 경우 `**typeof**` 를 사용하면 된다.

메서드의 반환 값에 타입을 만들 수도 있다.

```tsx
function getUserInfo(userId: string) {
  return {
    userId,
    name: 'choi',
    age: 1,
    height: 200,
    weight: 300,
    favoriteColor: 'black',
  }
}

type UserInfo = ReturnType<typeof getUserInfo>;
```

`ReturnType` 제너릭을 사용하면 된다. 함수의 값인 `getUserInfo` 가 아니라 **함수의 타입인 `typeof getUserInfo` 에 적용되었다.**

이런 기법(`Pick, Partial, ReturnType`)들은 적용대상이 값인지 타입인지 정확히 알고, 구분해서 처리해야 된다.

함수에서 매개변수로 매핑할 수 있는 값을 제한하기 위해 타입 시스템을 사용하는 것처럼, 제네릭 타입에서 매개변수를 제한할 수 있는 방법이 있다.

제네릭 타입에서 매개변수를 제한할 수 있는 방법은 바로 `extends` 를 사용하는 것이다. `extends`를 이용하면 제네릭 매개변수가 특정 타입을 확장한다고 선언 할 수 있다.

```tsx
interface User {
  firstName: string;
  lastName: string;
}

type DancingDuo<T extends User> = [T, T];

const couple1: DancingDuo<User> = [
  { firstName: 'foo1', lastName: 'bar1' },
  { firstName: 'foo2', lastName: 'bar2' },
] // O
 
const couple2: DancingDuo<{ firstName: string; }> = [ // TS2344: Type '{ firstName: string; }' does not satisfy the constraint 'User'. Property 'lastName' is missing in type '{ firstName: string; }' but required in type 'User'.
  { firstName: 'foo1', lastName: 'bar1' },
  { firstName: 'foo2', lastName: 'bar2' },
] // X

const couple3: DancingDuo<{ firstName: string; lastName: string; age: number; }> = [
  { firstName: 'foo1', lastName: 'bar1', age: 1 },
  { firstName: 'foo2', lastName: 'bar2', age: 2 },
] // O
```

`{ firstName: string; }` 는 User 타입을 확장하지 않기 때문에 에러가 발생한다.

`Pick` 제네릭의 정의는 `extends` 를 사용해서 완성할 수 있다.

```tsx
type Pick<T, K> = {
  [k in K]: T[k]; // TS2322: Type 'K' is not assignable to type 'string | number | symbol'.
}
```

K 는 T 와 무관하고 너무 범위가 넓다. K 는 실제로 T 의 키 값의 부분집합인 `keyof T` 가 되어야 한다.

```tsx
type Pick<T, K extends keyof T> = {
  [k in K]: T[k];
}
```

**타입이 값의 집합이라는 관점**에서 extends 는 확장이 아니라 부분집합이다.

값의 공간에서와 마찬가지로 반복적인 코드는 타입 공간에서도 좋지 않다. 타입들 간의 매핑을 위해 타입스크립트에서 제공해주는 도구들인 **keyof, typeof, 인덱스, 매핑된 타입들을 사용**하면 좋다.
