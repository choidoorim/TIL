# 아이템59. 타입스크립트 도입 전에 @ts-check 와 JSDoc 으로 시험해 보기

`@ts-check` 지시자를 통해서 타입스크립트 전환 시 어떤 문제가 발생하는지 미리 테스트해 볼 수 있다.

```jsx
// @ts-check
const person = {
  first: 'Grace',
  last: 'Hopper',
}
2 * person.first; // ~~~~ 산술 연산 오른쪽은 'any', 'number', 'bigint' 또는 열거형 형식이어야 합니다
```

`@ts-check` 덕분에 자바스크립트임에도 불구하고 타입체크가 동작했다. `@ts-check` 지시자를 통해서 몇 가지 오류들도 추가적으로 찾아낼 수 있다.

## 선언되지 않은 전역변수

변수를 선언할 때 보통 `let, const` 를 사용한다. 그러나 어딘가에 숨어있는 변수라면, 변수를 제대로 인식할 수 있도록 별도의 타입 선언 파일을 만들어야 한다.

```jsx
// types.d.ts 
interface UserData {
  firstName: string;
  lastName: string;
}
declare let user: UserData;

// @ts-check
// <reference path="./types.d.ts" />
console.log(user.firstName); // user is not defined
```

user 선언을 위해 `types.d.ts` 를 추가한다. 만약 선언 파일을 찾지 못하는 경우에는 트리플 슬래시 참조를 통해서 명시적으로 임포트할 수 있다.

## 알 수 없는 라이브러리

서드파티 라이브러리를 사용하고 있을 경우, 서드 파티 라이브러리의 타입 정보를 알아야 한다. `@ts-check` 를 사용하면 타입스크립트로 마이그레이션 하기 전에 서드파티 라이브러리들의 타입 선언을 활용해서 타입 체크를 테스트해 볼 수 있다.

## 부정확한 JSDoc

JSDoc 주석으로 타입 정보가 많이 담겨져 있다면 `@ts-check` 지시자만 간단히 추가함으로써 기존 코드에 타입 체커를 실험해볼 수 있고, 초기 오류를 빠르게 잡아낼 수 있다. 하지만 타입스크립트로 마이그레이션 시 중요한 점은 모든 코드가 주석이 아닌 타입스크립트 기반으로 전환되는 것을 잊지 말아야 한다.

```jsx
// @ts-check
/**
 * @param {number} val
 */
function double(val) {
  return 2 * val;
}
```
