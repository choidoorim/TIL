# 아이템42. 모르는 타입의 값에는 any 대신 unknown 을 사용하기

함수의 반환 값에 관련된 `unknown` 을 알아보자.

YAML 파서인 parseYAML 함수를 작성한다고 가정해보자.

```tsx
function parseYAML(yaml: string): any {
  //...
}
```

함수의 반환 값으로 `any` 타입을 사용하는 것은 좋지 않다. 대신 호출한 곳에서 반환 값을 원하는 타입으로 할당하는 것이 이상적이다.

```tsx
interface Book {
  name: string;
  author: string;
}

const book:Book = parseYAML(`
  name: Wuthering Heights
  author: Emily Brontë
`);
```

그러나 함수의 반환 값에 타입 선언을 강제할 수 없기 때문에, 호출한 곳에서 타입 선언을 생략하게 되면 `book` 변수는 암시적 `any` 타입이 되고 사용되는 곳마다 오류가 발생할 것이다.

```tsx
const book = parseYAML(`
  name: Wuthering Heights
  author: Emily Brontë
`);
alert(book.title); // 타입 체크시 오류 X, 런타임 오류 발생
book('read'); // 타입 체크시 오류 X, 런타임 시 'book is not a function; 에러 발생
```

따라서 `unknown` 을 반환하는 것이 좋다. `unknown` 을 이해하기 위해서는 할당 가능성의 관점에서 `any` 를 생각해볼 필요가 있다. `any` 가 강력하고 위험한 것은 2 가지 크게 2 가지 이유가 있다.

- 어떤 타입이든 any 타입에 할당 가능하다.
- any 타입은 어떤 타입으로도 할당 가능하다.

타입을 값의 집합으로 생각하는 관점에서 **한 집합은 다른 모든 집합의 부분 집합이면서 동시에 상위 집합이 될 수 없기** 때문에 any 타입은 타입 시스템과 다른 면이 있다. 이런 점이 `any` 가 강력하고 문제를 일으키는 원인이 된다. 즉, any 를 사용하게 되면 타입 체커는 무용지물이 된다.

`unknown` 은 `any` 대신에 쓸 수 있는 타입 시스템에 부합하는 타입이다. **`unknown` 타입은 값을 그대로 사용할 수 없기 때문에 타입 단언**을 해야 한다.

```tsx
interface Book {
  name: string;
  author: string;
}

const book = safeParseYAML(`
  name: Villette,
  author: Charlotte Bronte
`) as Book;
alert(book.title); // Property 'title' does not exist on type 'Book'.ts(2339)
```

**어떤 값이 있지만 값의 타입을 모를 때 `unknown` 을 사용**한다.

```tsx
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}
```

`unknown` 타입을 원하는 타입으로 바꾸는 방법중 타입 단언이 유일한 방법은 아니다. `instanceof` 를 체크한 뒤에 `unknown` 에서 원하는 타입으로 변환할 수 있다.

```tsx
function processValue(val: unknown) {
  if (val instanceof Date) {
    val; // val: Date
  }
}
```

또한 사용자 정의 타입 가드도 unknown 타입에서 원하는 타입으로 변환할 수 있다.

```tsx
interface Book {
  name: string;
  author: string;
}

function isBook(val: unknown): val is Book {
  return (
    typeof(val) === 'object' && val !== null && 
    'name' in val && 'author' in val
  )
}

function processValue(val: unknown) {
  if (isBook(val)) {
    val; // val: Book
  }
}
```

간혹 `unknown` 대신에 제네릭을 사용하는 경우도 있다.

```tsx
function safeParseYAML<T>(yaml: string): T {
  return safeParseYAML(yaml);
}
```

이 방법은 타입 단언문과 달라보이지만 동일한 기능을 한다.

```tsx
const foo = safeParseYAML<string>(''); // const foo: string
```

제네릭을 반환하기 보다는 `unknown` 을 반환하고 사용자가 직접 타입 단언을 사용하거나 타입을 알아서 좁히도록 강제하는 것이 더 좋은 방법이다.

이중 단언문을 사용할 경우 `any` 보다는 `unknown` 을 사용할 수도 있다.

```tsx
interface Foo {
  foo: string;
}

interface Bar {
  bar: string;
}

const foo: Foo = { foo: 'foo' };
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

`barAny` 와 `barUnk` 는 기능적으로 동일하지만, 나중에 단언문을 분리한다고 했을 때는 `unknown` 형태가 더 안전하다.

`any` 의 경우 분리될 경우 그 영향력이 전염병처럼 퍼지게 된다. 그러나 `unknown` 의 경우 분리될 경우 오류가 발생하기 때문에 더 안전한 것이다.

`unknown` 타입과 유사하지만 조금 다른 타입도 있다. `object` 또는 `{}` 이다.

```tsx
let foo: {} = 1;
let bar: {} = 'string';
let foo2: {} = true;
let bar2: {} = 100n;
```

`object` 또는 `{}` 를 사용하는 방법은 `unknown` 만큼 범위가 넓은 타입이지만, `unknown` 보다는 약간 좁다.

- `{}` 타입은 `**null` 과 `undefined` 를 제외한 모든 타입을 허용**한다.
- `object` 타입은 모든 비기본형 타입으로 이뤄진다. object 타입은 `number, boolean, string` 타입은 포함되지 않지만 **객체와 배열은 포함**된다.

`unknown` 타입이 도입되기 전에는 `{}` 를 더 많이 사용했다. 하지만 최근에는 `null` 과 `undefined` 가 불가능하다고 판단되는 경우만 `{}` 를 사용하고 대부분 `unknown` 타입을 사용한다.

`unknown` 타입을 사용하면 **사용자가 타입 단언문이나 타입 체크를 하도록 강제**할 수 있다.
