# 1. 타입스크립트와 자바스크립트 관계 이해하기

타입스크립트는 독특한 언어이다. 인터프리터(Python, Ruby…)로 실행되는 것도 아니며, 저수준 언어(C, Java…)로 컴파일 되는 것도 아니다. 자바스크립트로 컴파일되며, 실행 역시 타입스크립트가 아닌 자바스크립트로 실행된다.

타입스크립트는 문법적으로 자바스크립트의 상위 집합니다. 자바스크립트에 오류가 없다면 유효한 타입스크립트 프로그램이라고 할 수 있다. 그리고 타입스크립트는 코드를 파싱하고 자바스크립트로 변환할 수 있다.

자바스크립트는 파일 확장자를 `.js, .jsx` 를 사용하지만 타입스크립트는 `.ts, .tsx` 를 사용한다.

모든 자바스크립트 프로그램이 타입스크립트라는 것은 참이지만, 그 반대는 성립하지 않는다. 왜냐면 타입스크립트가 타입을 명시하는 추가적인 문법을 가지기 때문이다.

```tsx
// Typescript
function(who: string) {
	console.log('Hello', who);
}
```

```jsx
// Javascript
// Syntax Error 발생
function(who: string) {
	console.log('Hello', who);
}
```

위의 코드는 자바스크립트일 경우 에러가 발생한다. 왜냐하면 `: string` 은 타입스크립트 문법이기 때문이다.

그리고 타입스크립트는 타입을 지정하지 않아도 초기 값으로 타입을 추론한다.

```tsx
let age = 10; // age: number
```

타입 시스템의 핵심은 **런타임 환경에서 오류를 발생시킬 코드를 미리 찾아내는 것**이다. 그러나 타입 체커가 모든 것을 찾아내지는 않는다.

오류가 발생하지는 않지만 의도와 다르게 동작하는 코드들도 있다. 타입스크립트의 타입 체커는 추가적인 타입 구문 없이도 오류를 찾아낸다. 타입 구문 없이도 오류를 잡아낼 수 있지만 타입 구문을 추가한다면 더 많은 오류를 탐지할 수 있다.

```tsx
const states = [
  { name: 'Alabama', capital: 'Montgomery' },
  { name: 'Alaska', capitol: 'Juneau' }
  { name: 'Arizona', capital: 'Phoenix' },
]

for (const state of states) {
  console.log(state.capital);
}
```

예를 들어, 만약 타입 구문 없이 capital 을 실수로 capitol 로 바꿔도 타입스크립트는 오류를 찾지 못한다.

```tsx
interface State {
  name: string;
  capital: string;
}

const states: State[] = [
  { name: 'Alabama', capital: 'Montgomery' },
  { name: 'Alaska', capitol: 'Juneau' }
]

for (const state of states) {
  console.log(state.capital);
}
```

하지만 타입을 지정해준다면 오류를 찾을 수 있다.

![Untitled](https://user-images.githubusercontent.com/63203480/199016091-4721fef5-d86e-4697-a312-c5963edd3e4f.png)

타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 **모델링** 한다.

자바스크립트의 런타임 동작을 모델링하는 것은 타입스크립트 타입 시스템의 기본적인 원칙이다.

```tsx
const age_1 = 2 + '2';
const age_2 = '2' + 3;
console.log(age_1, age_2); // 22, 23
console.log(typeof age_1, typeof age_2); // string, string
```

이 코드는 다른 언어였다면 런타임 오류가 될만한 코드이다. 하지만 타입스크립트의 타입 체커는 정상으로 인식한다. 자바스크립트의 런타임 동작을 모델링하기 때문이다.

반대로 자바스크립트 런타임에서는 정상동작하는 코드에 오류로 표시하기도 한다.

```tsx
// Javascript
const a = null + 7; // 7
const b = [] + 12;  // '12'

// Typescript
const a = null + 7; // Error
const b = [] + 12;  // Error
```

이와 같이 언제 자바스크립트 런타임을 그대로 모델링할지, 또는 추가적인 타입 체크를 할지 불분명하다면 타입스크립트를 사용하는 것에 의문을 가질 수 있다.

작성된 프로그램이 타입 체커에서 통과되었다고 해도, 런타임에서 문제가 발생할 수 있다.

```tsx
const names = ['Alice', 'Bob'];
console.log(names[2].toUpperCase());
// 런타임 Error
// cannot read properties of undefined (reading 'tolowercase')
```

즉, 타입스크립트는 런타임에서 문제가 발생할 코드를 찾아내려고 하지만 **모든 것을 찾아낼 수 없기에 절대 기대하면 안된다**.