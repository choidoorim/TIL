# 아이템19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트에서 많은 타입 구문은 사실 불필요하다.

```tsx
let x: number = 12;
```

사실 아래와 같이해도 충분하다.

```tsx
let x = 12; // let x: number;
```

편집기에서 마우스를 올려보면 타입이 이미 number 로 추론되어 있는 것을 확인할 수 있다. 타입추론이 된다면 명시적 타입 구문은 오히려 방해가 될 뿐이다.

```tsx
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  }
  died: {
    where: string;
  }
} = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1997',
  },
  died: {
    where: 'Battle Creek, MI',
  }
}
```

이 코드는 타입을 생략하고 사용해도 된다.

```tsx
const person = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1997',
  },
  died: {
    where: 'Battle Creek, MI',
  }
}
```

이처럼 추가로 타입을 작성하는 것은 거추장스러울 뿐이다.

배열도 객체와 마찬가지이다. 타입스크립트는 입력을 받아 연산을 하는 함수가 어떤 타입을 반환하는지 정확히 알고 있다.

```tsx
function square(nums: number[]) {
  return nums.map(x => x * x);
}

const squares = square([1, 2, 3, 4]); // const squares: number[];
```

타입스크립트는 생각보다 더 정확하게 추론을 한다.

```tsx
const axis1: string = 'x';
const axis2 = 'y'; // const axis2: "y"
```

`axis2` 변수를 `string` 으로 예상하기 쉽지만 **실제로 타입스크립트는 `y` 로 추론**한다.

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

위와 같은 코드는 비구조화 할당문(destructuring assignment))으로 통해 구현하는 것이 훨씬 더 좋다.

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

타입스크립트에서 변수의 타입은 일반적으로 처음 등장할 때 결정된다. 이상적인 타입스크립트 코드는 **함수(메서드) 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역 변수에는 타입구문을 넣지 않는다**.

- 메서드 시그니처: 함수의 선언부에 명시되는 매개변수 리스트

타입 구문을 생략하여 방해되는 것들을 최소화하고 코드를 읽는 사람이 **구현로직에 집중**할 수 있게 하는 것이 좋다.

함수 매개변수에 타입 구문을 생략하는 경우도 간혹있는데 매개변수에 **기본값이 있는 경우**이다.

```tsx
function parseNumber(str: string, base = 10) {
  //...
}
```

여기서 기본 값이 10이기 때문에 `base` 의 타입은 `number` 로 추론된다.

타입이 추론될 수 있어도 타입을 명시하고 싶은 상황이 존재한다.

### 1. **객체 리터럴을 정의**할 때

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}

const elmo: Product = {
  name: 'Tickle Me Elmo',
  id: '04818862', // TS2322: Type 'string' is not assignable to type 'number'.
  price: 28.99,
}
```

위와 같은 정의에 타입을 명시하면, 잉여 속성 체크가 동작한다. 잉여 속성 체크는 특히 **선택적 속성이 있는 타입의 오타 같은 오류를 잡는데 효과적**이다.

만약 타입구문을 제거한다면 잉여 속성 체크가 동작하지 않고, 객체를 선언하는 곳이 아닌 객체가 사용되는 곳에서 타입 오류가 발생하게 된다.

### 2. 함수의 반환타입

타입 추론이 가능하더라도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는 것이 좋다. 반환 타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않는다.

반환 타입을 명시하면 함수에 대해 더욱 명확하게 알 수 있다. 미리 타입을 명시하는 방법은, 함수를 구현하기 전에 테스트를 먼저 작성하는 TDD 와 비슷하다.

그리고 명명된 타입을 사용하기 위해서 함수의 반환 타입을 설정해줘야 한다.

```tsx
interface Vector2D { x: number, y: number }
function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
// function add(     
//   a: Vector2D,     
//   b: Vector2D): {x: number, y: number}
```

위 코드에서 a, b 의 타입은 명명된 타입인 Vector2D 로 추론되지만, 반환 타입은 그렇지 않다. 반환 타입을 명시하면 더욱 직관적인 표현을 할 수 있다.
