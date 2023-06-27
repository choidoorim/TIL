# 아이템17. 변경 관련된 오류 방지를 위해 readonly 사용하기

삼각수를 출력하는 코드가 있다.

```tsx
function printTriangles(n: number) {
  const nums = [];
  for (let i = 0; i < n; i++) {
    nums.push(i);
  }
  return arraySum(nums);
}

function arraySum(arr: number[]) {
  let sum = 0, num;
  while((num = arr.pop()) !== undefined) {
    sum += num;
  }
  return sum;
}
```

arraySum 함수는 배열안에 있는 모든 수를 합친다. 그런데 계산이 끝나면 `printTriangles` 함수의 `nums` 배열이 전부 빈 상태가 된다. 자바스크립트 배열은 내용을 변경할 수 있기 때문에 타입스크립트에서도 역시 오류없이 통과하게 된다.

오류의 범위를 좁히기 위해서 `arraySum` 함수가 **배열을 변경시키지 않는다는 선언을 `readonly` 접근 제어자를 통해서 할 수 있다**.

```tsx
//...
function arraySum(arr: readonly number[]) {
  let sum = 0, num;
  while((num = arr.pop()) !== undefined) { // TS2339: Property 'pop' does not exist on type 'readonly number[]'.
    sum += num;
  }
  return sum;
}
```

`readonly number[]` 는 타입이고, `number[]` 와 구분되는 몇가지 특징이 있다.

- 배열의 요소를 읽을 수는 있지만, 쓰지는 못한다.
- length 를 읽을 수는 있지만, 배열을 변경해서 바꿀 수는 없다.
- 배열을 변경하는 메서드들을 사용할 수 없다.

`**number[]` 는 `readonly number[]` 보다 기능이 많기 때문에 `readonly number[]` 의 서브타입**이 된다. 따라서 변경 가능한 배열(`numbers[]`)을 readonly 배열(`readonly numbers[]`)에 할당하는 것은 가능하다. 하지만 반대로 readonly 배열을 변경 가능한 배열에 할당하는 것은 안된다.

```tsx
// O
const nums: number[] = [1, 2, 3, 4];
const readonlyNums: readonly number[] = nums;

// X - TS4104: The type 'readonly number[]' is 'readonly' and cannot be assigned to the mutable type 'number[]'.
const readonlyNums: readonly number[] = [1, 2, 3, 4];
const nums: number[] = readonlyNums;
```

매개변수를 readonly 로 설정하면 아래와 같은 장점들이 있다.

- 타입스크립트는 **매개변수가 함수내부에서 변경이 일어나는지 체크**를 한다.
- **호출하는 쪽에서는 함수가 매개변수를 변경시키지 않는다는 보장**을 받을 수 있게 된다.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

자바스크립트(당연 타입스크립트도)는 명시적으로 언급하지 않는다면 함수가 매개변수를 변경하지 않는다고 가정한다. 이렇게 암묵적인 방식은 타입체크에 문제를 일으킬 수 있기 때문에 **명시적인 방법을 사용하는 것이 컴파일러와 사람에게 모두 좋다**.

매개변수가 readonly 로 선언되지 않은 함수를 호출하는 경우도 있다. 다른 라이브러리에 있는 함수를 호출하는 경우네는 타입 선언을 바꿀 수 없기 때문에 타입 단언문으로 사용해야 한다.

**readonly 는 얕게(shallow) 동작한다는 것에 유의하면서 사용**해야 한다.

```tsx
const dates: readonly Date[] = [new Date()];
dates.push(new Date()); // 에러 - TS2339: Property 'push' does not exist on type 'readonly Date[]'.
dates[0].setFullYear(2037); // 정상

const o: Readonly<Outer> = { inner: { x: 0 } };
o.inner = { x: 1 }; // 에러 - TS2540: Cannot assign to 'inner' because it is a read-only property.
o.inner.x = 1; // 정상

type Outer2 = Readonly<Outer>; // {readonly inner: {x: number}}
```

타입 별칭(Type Alias) 를 만들어서 확인해보면 **readonly 접근제어자는 inner 에 적용되는 것이지, x 는 포함되지 않는다는 것**이다.

Deep Readonly 는 기본적으로 제공되지는 않지만 제네릭으로 구현이 가능하다. 하지만 구현이 까다롭기 때문에 `ts-essentials`에 있는 `DeepReadonly` 제네릭처럼 라이브러리를 사용하는 것이 좋다.

인덱스 시그니처에도 readonly 를 사용해서 읽기는 허용하고 쓰기를 방지할 수도 있다.

```tsx
interface User {
  name: string;
  age: number;
}

type ReadonlyUser<T extends User> = {
  readonly [k in keyof T]: T[k];
}

const user: ReadonlyUser<User> = {
  name: 'name',
  age: 1,
}
```

인덱스 시그니처에 readonly 를 사용하면 객체의 속성이 변경되는 것을 방지할 수 있다.
