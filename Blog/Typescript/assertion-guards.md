# Typescript - Assertion Function

- 아래 모든 예제 코드들은 [Typescript Playground](https://www.typescriptlang.org/play) 에서 작성했습니다.

유저가 존재하는지에 대한 여부를 확인하는 로직이 아래와 같다고 가정해봅시다.

```tsx
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

declare const user: User | null;

function validateNotNullUser(user: User | null) {
  if (!user) {
    throw new Error('유저가 존재하지 않습니다');
  }
}

validateNotNullUser(user);
user; // User | null
```

`validateNotNullUser` 함수는 유저가 존재하지 않을 경우 에러를 발생시킵니다. `validateNotNullUser` 함수 실행 후에 `user` 변수의 타입은 어떻게 되어야할까요? 당연히 `User` 일 것이라고 예상됩니다. 하지만 실제로 확인해보면 여전히 `user` 변수의 타입은 `User | null` 인 것을 확인할 수 있습니다.

만약 함수를 사용하지 않고 바로 유저 존재여부를 확인한다면 위와 같은 문제가 발생하지 않습니다.

```tsx
if (!user) {
  throw new Error('유저가 존재하지 않습니다');
}
user; // User
```

하지만 유저 존재여부를 체크하는 로직 등은 중복되는 경우가 빈번하여 재사용될 가능성이 높기 때문에 별도의 함수로 나누는 것이 좋다고 생각했습니다.

타입스크립트에서는 [User-Defined Type Guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html) 라는 개념이 있습니다. “**어떤 인자명은 어떠한 타입이다”** 라는 값을 반환하는 함수입니다. 하지만 위의 문제는 유저가 존재하지 않을 경우 값을 반환하는 것이 아니라 에러를 발생시키고 있기 때문에 User-Defined Type Guards 는 활용할 수 없을 것 같습니다.

그렇다면 어떻게 위 문제를 해결할 수 있을까요? 타입스크립트에서는 함수의 매개변수가 어떤 타입인지를 보장할 수 있도록 해주는 방법이 있다고 합니다. 함수의 반환 타입에 `asserts <parameter> is <type>` 를 추가해주면 됩니다.

```tsx
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

declare const user: User | null;

function validateNotNullUser(user: User | null): asserts user is User {
  if (!user) {
    throw new Error('유저가 존재하지 않습니다');
  }
}

validateNotNullUser(user);
user; // User
```

추가 후 `validateNotNullUser` 함수를 실행하면 `user` 변수의 타입이 `User` 가 된 것을 확인할 수 있습니다.  `is` 키워드를 제거하고 사용할 수도 있습니다. `is` 키워드를 제거한다는 것은 매개변수로 전달되는 모든 것들이 참이어야 한다고 정의하는 것이라고 합니다.

```tsx
function validateNotNullUser(user: User | null): asserts user {
  if (!user) {
    throw new Error('유저가 존재하지 않습니다');
  }
}

validateNotNullUser(user);
user; // User
```

문제는 해결됐지만 한가지 걱정되는 점이 있습니다. `asserts <parameter> is <type>` 은 결국 타입 단언과 같은 기능을 합니다. 아래 예제들은 실제로 예상되는 로직과 타입이 일치하지 않고 있습니다.

```tsx
// EX1
function validateNotNullUser(user: User | null): asserts user is null {
  if (!user) {
    throw new Error('유저가 존재하지 않습니다');
  }
}

validateNotNullUser(user);
user; // type - null(O), User(X)

// EX2
function validateNotNullUser(user: User | null): asserts user {
  if (user) {
    throw new Error('유저가 존재합니다');
  }
}

validateNotNullUser(user);
user; // type - User(O), null(X)
```

하지만 이러한 문제를 고려하더라도 위와 같은 방법으로 중복되는 로직들을 처리하여 재사용성을 높일 수 있기 때문에 개인적으로 단점보다는 장점이 더 크다는 생각이 듭니다.

참고

https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions

https://blog.logrocket.com/assertion-functions-typescript/
