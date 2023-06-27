# 아이템15. 동적인 데이터에 인덱스 시그니처 사용하기

자바스크립트의 장점중 하나는 객체를 쉽게 생성할 수 있다는 것이다. 자바스크립트 객체는 문자열 키가 타입의 값에 관계없이 매핑된다.

```tsx
const rocket = {
  name: 'Falcon 9',
  variant: 'Block 5',
  thrust: '7,607 kN',
}
```

타입스크립트에서는 타입에 인덱스 시그니처(`{ [key in string]: string }`)를 명시해서 유연하게 매핑을 할 수 있다.

```tsx
type Rocket = {
  [key in string]: string;
}

const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'Block 5',
  thrust: '7,607 kN',
}
```

- 키의 이름(`key`): 키의 위치만 표시하는 용도로 타입 체커에서는사용하지 않는다.
- 키의 타입(`in string`): string 또는 number 와 symbol 의 조합이어야 한다. 보통은 string 을 사용한다.
- 값의 타입(`string`): 어떤 것이든 될 수 있다(`number, string, object, interface, type…`).

하지만 이렇게 타입체크를 하게 되면 몇 가지 단점이 생긴다.

- 모든 키를 허용한다. name 대신 Name 으로 작성해도 Rocket 의 타입이 된다.
- 키 마다 다른 타입을 가질 수 없다. `string` 타입일 경우 모든 키에 대한 타입이 `string` 만 허용하게 되는 것이다.
- 타입스크립트 서비스에서 제공하는 자동완성 기능을 제공받지 못한다.

이처럼 시그니처는 부정확하기 때문에 인터페이스를 사용하는게 더 좋다.

```tsx
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}

const falconHeavy: Rocket = {
  name: 'Falcon',
  variant: 'v1',
  thrust_kN: 15_200,
}
```

따라서 타입스크립트 서비스에서 제공하는 자동완성, 정의로 이동, 이름 바꾸기 등이 모두 사용할 수 있고 동작한다.

인덱스 시그니처는 **동적 데이터를 표현할 때 사용**한다. 예를 들어 CSV 파일처럼 헤더 행에 열 이름이 있고, **데이터 행을 열 이름과 값으로 매핑하는 객체로 나타내고 싶은 경우**이다. **즉, 런타임 때까지 객체의 속성을 알 수 없는 경우에만 사용**해야 한다.

```tsx
function parseCSV(input: string): {[columnName: string]: string}[] {
  //...
}
```

그리고 안전한 접근을 위해 **인덱스 시그니처의 값 타입에 undefined 를 추가하는 방법**도 좋은 고려사항이다.

```tsx
function safeCSV(): {[columnName: string]: string | undefined}[] {
  //...
}
```

타입에 대한 필드들이 제한적인 상황이라면 인덱스 시그니처로 모델링하면 안된다. 오히려 선택적 필드나 유니온 타입으로 모델링하는 것이 좋다.

```tsx
// 광범위
interface Row1 {
  [column: string]: number;
}
// 선택적 필드
interface Row2 {
  a: number;
  b?: number;
  c?: number;
  d?: number;
}
// 유니온 타입 - 가장 정확, 하지만 사용하기 번거로움
type Row3 = 
  | { a: number; }
  | { a: number; b: number; }
  | { a: number; b: number; c: number }
  | { a: number; b: number; c: number; d: number }
```

더 나은 방법도 있다. 바로 `Record` 와 매핑된 타입을 사용하는 것이다.

```tsx
type Vec3D = Record<'x' | 'y' | 'z', number>;
```

`Record` 는 키 타입에 유연성을 제공해주는 제네릭 타입이다.

```tsx
type Vec3D = {[k in 'x' | 'y' | 'z']: number};
// type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }
type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number };
// type ABC = {
//   a: number;
//   b: number;
//   c: number;
// }
```

매핑된 타입은 키마다 별도의 타입을 사용하게 해준다. 또한 조건부 타입(`?`)을 사용할 수도 있다.

즉, 가능하다면 인덱스 시그니처보다 Record 나 매핑된 타입을 사용해서 정확한 타입을 사용하는 것이 좋다.
