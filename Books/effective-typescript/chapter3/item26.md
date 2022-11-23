# 아이템26. 타입추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 추론할 때 단순히 값만 고려하지는 않는다. **값이 존재하는 곳의 문맥**까지도 살핀다.

그런데 문맥을 통해 타입을 추론하면 가끔 이상한 결과가 나오기도 한다. 하지만 이럴 경우 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있다.

자바스크립트는 동작과 실행 순서를 바꾸지 않고 표현식을 상수로 분리할 수 있다.

```tsx
function setLanguage(language: string) {}

setLanguage('JavaScript');

let language = 'JavaScript';
setLanguage(language);
```

만약 문자열(`string`) 타입을 더 명확하게 리터널 타입의 유니온으로 바꿔보겠습니다.

```tsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) {}

setLanguage('JavaScript');

let language = 'JavaScript';
// ERROR
// TS2345: Argument of type 'string' is not assignable to parameter of type 'Language'.
setLanguage(language); 
```

변수로 값을 분리한다면 변수(`language`)에 할당하는 시점에 타입스크립트는 타입 추론을 한다. `string` 타입으로 추론했고, `string` 타입은 `Language`(유니온 타입) 에 할당할 수 없기 때문에 오류가 발생했다.

이러한 문제는 2 가지 방법으로 해결할 수 있다.

**1. 변수에 타입 선언**

```tsx
let language: Language = 'JavaScript';
setLanguage(language);
```

만약 변수에 값을 **재할당 해야 한다면 타입 선언**으로 사용하는 것이 좋다.

**2. 상수(`const`)로 변경**

```tsx
const language = 'JavaScript';
setLanguage(language);
```

만약 값의 재할당이 필요하지 않다면 상수로 선언하여 language 변수를 더 정확한 타입인 `JavaScript` 로 선언할 수 있다.

이와 같은 추론방식은 number 타입과 유니온 타입간에도 동일하게 발생한다.

## 튜플 사용 시 주의점

튜플도 위와 같은 문제가 발생한다.

```tsx
function panTo(where: [number, number]) {}

panTo([10, 20]);

const loc = [10, 20]; // type: number[]
// TS2345: Argument of type 'number[]' is not assignable to parameter of type '[number, number]'. Target requires 2 element(s) but source may have fewer.
panTo(loc);
```

`number[]` 타입은 길이를 알 수 없는 배열이라 대부분의 많은 배열이 `[number, number]` 와 맞지 않는 수의 요소들을 가지게 되기 때문에 할당할 수 없다는 오류가 발생한다. loc 변수는 이미 const 로 선언했기 때문에 상수 선언으로는 해결할 수 없다.

**1. 타입 선언**

```tsx
function panTo(where: [number, number]) {}

panTo([10, 20]);

const loc: [number, number] = [10, 20];
panTo(loc);
```

**2. 상수 문맥 제공**

```tsx
function panTo(where: [number, number]) {}

panTo([10, 20]);

const loc = [10, 20] as const;
// TS2345: Argument of type 'readonly [10, 20]' is not assignable to parameter of type '[number, number]'. The type 'readonly [10, 20]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
panTo(loc);
```

`const` 는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow) 상수인 반면에, `as const` 는 그 값이 내부(deeply)까지 상수라는 것을 타입스크립트에게 알려준다.

하지만 추론된 타입을 확인해보면 `readonly [10, 20]` 으로 너무 과하게 정확하게 추론이 되어서 오류가 발생한다. 이런 상황에서 최선의 해결책은 `panTo` 함수에 `readonly` 구문을 추가하는 것이다.

```tsx
function panTo(where: readonly [number, number]) {}

panTo([10, 20]);

const loc = [10, 20] as const;
panTo(loc);
```

`as const` 는 문맥에 관련된 문제를 해결할 수 있지만 한 가지 단점을 가지고 있다.

```tsx
function panTo(where: readonly [number, number]) {}

const loc = [10, 20, 30] as const; // ERROR(X)
panTo(loc); // ERROR(O)
```

만약 타입 정의에 실수가 있을 경우 오류는 타입을 정의한 곳이 아닌 호출하는 곳에서 발생하게 된다.
