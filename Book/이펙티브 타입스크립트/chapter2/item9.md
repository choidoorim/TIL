# 아이템9. 타입 단언보다는 타입 선언을 사용하기

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 2 가지이다.

```tsx
interface Person {
  name: string;
}

const alice: Person = { name: 'Alice' }; // 타입선언
const bob = { name: 'Bob' } as Person;   // 타입단언
```

타입 선언은 할당되는 값이 해당 타입을 만족하는지 검사한다. 하지만 타입 단언은 강제로 타입을 지정했기 때문에 타입 체커에서 오류를 무시하라고 해서 오류를 발생시키지 않는다.

```tsx
interface Person {
  name: string;
}

const alice: Person = { name: 'Alice' };
const bob = {} as Person; // 오류 X
```

이와 같은 문제는 속성을 추가할 때도 발생한다.

```tsx
interface Person {
  name: string;
}

// ERROR - TS2322: Type '{ name: string; age: number; }' is not assignable to type 'Person'. Object literal may only specify known properties, and 'age' does not exist in type 'Person'.
const alice: Person = { name: 'Alice', age: 11 };
const bob = { name: 'Alice', age: 11 } as Person;
```

타입 선언에는 속성 체크를 통해 에러가 발생했지만, 타입 단언에는 아무일도 발생하지 않는다.

따라서 타입 단언이 꼭 필요한 경우가 아니라면 타입 선언을 사용하는 것이 좋다.

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있다.

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({name}));
// type - const people: {name: string}[]
```

이럴 경우 타입 단언을 사용하면 문제가 해결되기는 하지만 위의 예제처럼 문제를 잡지못하는 일이 발생한다.

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({name}) as Person);
// type - Person[]

const people = ['alice', 'bob', 'jan'].map(name => ({}) as Person); // 오류 X
```

따라서 타입 선언을 통해 타입을 정의해주는 것이 좋다.

```tsx
// 반환 타입 명시
const people = ['alice', 'bob', 'jan'].map((name: string): Person => ({name}));
// 최종적으로 원하는 타입을 명시
const people: Person[] = ['alice', 'bob', 'jan'].map(name => ({name}));
```

그러나 함수 호출 체이닝이 연속된 곳에서는 체이닝이 시작에서부터 명명된 타입을 가져야 한다.
