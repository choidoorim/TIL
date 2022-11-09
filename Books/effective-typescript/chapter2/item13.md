# 아이템13. 타입과 인터페이스의 차이점 알기

타입스크립트에서 Named Type(명명된 타입)은 2 가지가 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}

interface IState {
  name: string;
  capital: string;
}
```

클래스를 사용해서도 가능하지만 클래스는 값으로도 쓰일 수 있는 자바스크립트 런타임의 개념이다.

대부분의 경우에는 타입과 인터페이스를 선택해서 사용해도 된다. 하지만 두 차이를 알고, 같은 상황에서는 동일한 방법으로 명명된 타입(Named Type) 을 정의해서 일관성을 유지해야 한다.

- **참고**: 타입에 I(인터페이스), T(타입) 라는 접두사를 붙이는 것은 현재 지양해야 되는 스타일로 여겨진다. 예제에서만 사용!

### 비슷한점

**1. 인덱스 시그니처**

인덱스 시그니처는 타입과 인터페이스 모두 사용할 수 있다.

```tsx
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

**2. 함수 타입 정의**

또한 함수 타입도 정의가 가능하다.

```tsx
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = x => '' + x;
const toStrI: IFn = x => '' + x;
```

자바스크립트에서 함수는 호출가능한 객체이기 때문에 아래와 같은 코드도 가능하다.

```tsx
type TFnWithProperties = {
  (x: number): number;
  prop: string;
}

interface IFnWithProperties {
  (x: number): number;
  prop: string;
}
```

**3. 제너릭 타입**

타입과 인터페이스 모두 제너릭이 가능하다.

```tsx
type TPair<T> = {
  first: T,
  second: T;
}

interface IPair<T> {
  first: T,
  second: T;
} 
```

**4. 확장**

인터페이스는 타입을 확장할 수 있고, 타입은 인터페이스를 확장할 수 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}

interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop  = IState & { population: number };
```

주의 해야할 점은 **인터페이스는 유니온 타입과 같은 복잡한 타입을 확장하지 못한다는 것**이다.

복잡한 타입의 확장을 위해서는 타입(`type`)과 `&` 를 사용해야 한다.

**5 . 클래스 구현(`implements`)**

클래스 구현은 타입과 인터페이스 모두 사용할 수 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}

class StateT implements TState {
  name: string = '';
  capital: string = '';
}
class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

### 다른 점

**1. 유니온 타입**

유니온 타입은 있지만 유니온 인터페이스는 없다.

```tsx
type AorB = 'a' | 'b';
```

인터페이스는 타입을 확장할 수는 있지만 유니온은 할 수 없다.

별도의 2개 이상의 타입을 하나의 변수로 매핑하는 인터페이스를 만들 수 있다.

```tsx
type Input = { /* ... */ };
type Output = { /* ... */ };

interface VariableMap {
  [name: string]: Input | Output;
}
```

또는 유니온 타입에 속성을 추가한 타입을 만들 수도 있다.

```tsx
type NamedVariable = (Input | Output) & { name: string };
```

이러한 타입은 인터페이스로 구현할 수 없다. 보통 타입은 인터페이스보다 쓰임새가 다양하다.

튜블과 배열 타입도 type 키워드를 통해 더 간결하게 표현할 수 있다.

```tsx
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
const namedNums: NamedNums = ['asd', 1, 2, 3];
```

인터페이스로도 튜플과 비슷하게 구현할 수 있기는 하다.

```tsx
interface Tuple {
  0: number;
  1: number;
  length: 2;
}

const t: Tuple = [10, 20];
```

하지만 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드를 사용할 수 없다. 그러므로 **튜플은 타입으로 구현**하는게 더 좋다.

**2. 확장,보강(Augment)하는 방법**

인터페이스가 가지고 있는 타입에 없는 몇 가지 기능 중 하나는, 보강 기법을 사용할 수 있다는 것이다.

```tsx
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000,
}
```

위의 예제처럼 속성을 확장하는 것을 **선언 병합(declaration merging)** 이라고 한다. 이처럼 선언병합을 사용하기 위해서는 반드시 인터페이스를 사용해야 한다.

타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합한다.

예로, Array 인터페이스는 `lib.es5.d.ts` 에 선언된 인터페이스가 사용된다. 그러나 `tsconfig.json`의 lib 목록에 ES2015를 추가하면 타입스크립트는 `lib.es2015.d.ts`에 선언된 인터페이스를 병합한다. 이처럼 병합을 통해 다른 Array 인터페이스에 추가된다.

타입은 기존타입에 추가적인 보강이 없는 경우에만 사용해야 한다.

### 결론

**복잡한 타입을 가져야 한다면 무조건 타입 별칭(Type Alias) 를 사용**해야 한다.

그러나 타입과 인터페이스 2 가지로 표현이 가능하다면 일관성과 보강의 관점에서 봐야 한다. 인터페이스를 사용하는 코드베이스일 경우 인터페이스를 사용하고, 타입을 사용하는 코드베이스일 경우 타입을 사용하면 된다.

어떤 **API에 대한 타입을 작성해야 한다면 인터페이스**가 좋다. API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문이다.

그러나 **프로젝트 내부적으로 사용되는 타입에는 선언병합이 발생할 수 있기 때문에 이럴 때는 타입을 사용**하는 것이 좋다.
