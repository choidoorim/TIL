# 아이템58. 모던 자바스크립트로 작성하기

깃허브, 에어비앤비에서도 각각 15%, 38% 의 자바스크립트 에러는 타입스크립트로 작성했을 경우 조기에 발견하고 수정할 수 있었던 것들이라고 한다. 프로젝트에 타입스크립트를 사용하기로 했다면, 점진적으로 마이그레이션하고 실행 및 테스트를 해야 한다. 만약 대규모 프로젝트일 경우 점진적으로 전환해야 한다.

타입스크립트는 타입 체크 기능과 타입스크립트 코드를 특정 자바스크립트 버전으로 컴파일할 수 있는 기능도 가지고 있다. 즉, **타입스크립트 컴파일러를 자바스크립트 트랜스파일러로 사용**할 수 있다. 타입스크립트는 자바스크립트의 상위 집합이기 때문에 최신 버전의 자바스크립트를 옛날 버전의 자바스크립트 코드로 변경이 가능하다.

옛날 버전의 자바스크립트 코드를 타입스크립트 컴파일러에서 동작하게 만든다면 이후로는 최신버전의 자바스크립트 기능을 추가해도 문제가 없다. 즉, 옛날 자바스크립트 코드를 최신 자바스크립트로 바꾸는 작업은 타입스크립트로 전환하는 작업의 일부로 볼 수 있다.

모던 자바스크립트의 새로운 기능들은 모두 좋지만, **타입스크립트를 도입할 때 가장 중요한 기능은 ECMAScript 모듈과 ES2015 클래스**이다.

## ECMAScript 모듈 사용하기

**ES2015 부터는 import 와 export 를 사용하는 ECMAScript 모듈이 표준**이 되었다. 만약 마이그레이션을 할 자바스크립트가 단일 파일이거나 비표준 모듈 시스템을 사용 중이라면 ES 모듈로 전환하는 것이 좋다.

그리고 ES 모듈 시스템을 사용하기 위해서 프로젝트의 종류에 따라 웹팩(Webpack) 이나 ts-node 같은 도구가 필요한 경우도 있다.

```jsx
// a.js
const b = require('./b');

console.log(b.name);

// b.js
const name = 'Module B';
module.exports = { name };
```

위 기능을 ES 모듈로 바꾸면 아래와 같다.

```tsx
// ECMAScript module
import * as b from './b';
console.log(b.name);

export const name = 'Module B';
```

## 프로토타입 대신 클래스 사용하기

과거에 자바스크립트에서는 프로토타입 기반의 객체 모델을 사용했다. 그러나 많은 개발자들이 클래스 기반 모델을 더 선호했기 때문에 결국 ES2015 에 class 를 사용하는 클래스 기반 모델이 도입됐다.

마이그레이션 하려는 코드에서 단순한 객체를 다룰 때 프로토타입을 사용하고 있었다면 클래스로 바꾸는 것이 좋다.

```jsx
function Person (first, last) {
  this.first = first;
  this.last = last;
}

Person.prototype.getName = function () {
  return this.first + ' ' + this.last;
}

const marie = new Person('Marie', 'Curie');
const personName = marie.getName();
```

프로토타입 기반 객체를 클래스 기반으로 바꾸면 아래와 같다.

```tsx
class Person {
  first: string;
  last: string;

  constructor(first: string, last: string) {
    this.first = first;
    this.last = last;
  }

  getName() {
    return this.first + ' ' + this.last;
  }
}

const marie = new Person('Marie', 'Curie');
const personName = person.getName();
```

프로토타입으로 구현한 객체보다 클래스로 구현한 객체가 가독성이 훨씬 뛰어나고 문법이 간결하고 직관적이다.

## var 대신 let/const 사용하기

자바스크립트의 `var` 은 스코프 규칙에 문제가 있다. `let` 과 `const` 는 제대로 된 블록 스코프 규칙을 가지고, 일반적으로 우리가 아는 방식으로 동작한다.

만약 var 를 let, const 로 변경하면, 일부 타입스크립트 코드에서 오류를 표시할 수 있다. 오류가 발생한 부분은 잠재적으로 스코프 문제가 존재하는 것이기에 수정해야 한다.

중첩된 함수 구문에서도 var 의 경우와 비슷한 스코프 문제가 존재한다.

```jsx
function foo() {
  bar();
  function bar() {
    console.log('hello');
  }
}

foo();
```

foo 함수를 호출하면 bar 함수의 정의가 호이스팅되어 가장 먼저 수행되기 때문에 bar 함수가 문제없이 호출되고 `"hello"` 가 출력된다. 호이스팅은 실행 순서를 예상하기 어렵게 만들고 직관적이지 않다. 대신 함수 표현식(`const bar = () => { ... }`)을 사용해서 호이스팅 문제를 피하는 것이 좋다.

## for(;;) 대신 for-of 또는 배열 함수 사용하기

과거 자바스크립트에는 배열을 순회할 때 C 스타일의 `for` 루프를 사용했다.

```jsx
for (var i = 0; i < array.length; i++) {
  const el = array[i];
  //..
}
```

모던 자바스크립트에서는 `for-of` 가 존재한다.

```jsx
for (el of array) {
  // ...
}
```

`for-of` 는 코드가 간결하고 `index` 를 사용하지 않기 때문에 실수를 줄일 수 있다. 인덱스 변수가 필요한 경우에는 `forEach` 메서드를 사용하면 된다.

```jsx
array.forEach((el, i) => {
  //...
});
```

`for-in` 문법도 존재하지만 몇 가지 문제점이 있기 때문에 사용하지 않는 것이 좋다.

```jsx
const arr = [1, 2, 3];
const keys = Object.keys(arr); // type -> string[]
for (const index in arr) {
  console.log(typeof index); // string
	console.log(index); // 0, 1, 2
  const x = arr[index]; // type -> number
}
```

## 함수 표현식보다 화살표 함수 사용하기

`this` 키워드는 일반적인 변수와는 다른 스코프 규칙을 가지고 있기에, 자바스크립트에서 가장 어려운 개념 중 하나이다. 일반적으로는 this 가 클래스 인스턴스를 참조하는 것을 기대하지만 예상치 못한 결과가 나올 수 있다.

```jsx
class Foo {
  method() {
    console.log(this);
    [1, 2].forEach(function(i) {
      console.log(this);
    })
  }
}

const f = new Foo();
f.method(); // Foo, undefined, undefined
```

대신 화살표 함수를 사용한다면 상위 스코프의 this 를 유지할 수 있다.

```jsx
class Foo {
  method() {
    console.log(this);
    [1, 2].forEach((i) => {
      console.log(this);
    })
  }
}

const f = new Foo();
f.method(); // Foo, Foo, Foo
```

그리고 컴파일러 옵션에 `nolmplicitThis`(or `strict`) 옵션을 활성화(`true`) 한다면, 타입스크립트가 this 바인딩 오류를 표시해주므로 설정하는 것이 좋다.

## 단축 객체 표현(Compact Object Literal)과 구조 분해(**Destructuring**) 할당 사용하기

```jsx
const x = 1, y = 2, z = 3;
const pt = {
  x: x,
  y: y,
  z: z,
};
```

변수와 객체 속성의 이름이 같다면, 간단하게 아래와 같이 작성할 수 있다.

```tsx
const x = 1, y = 2, z = 3;
const pt = { x, y, z };
console.log(pt); // { x: 1, y: 2, z: 3 }
```

후자의 코드가 중복된 이름을 생략하고 간결하기 때문에 실수가 적고 가독성이 좋다.

화살표 함수 내에서 객체를 반환할 때는 소괄호로 감싸야 한다.

```tsx
const pt = ['A', 'B', 'C'].map((char, index) => ({ char, index }));
console.log(pt); // [{ char: 'A', index: 0 }, { char: 'B', index: 1 }, { char: 'C', index: 2 }]
```

함수의 구현부에는 블록이나 단일 표현식이 필요하기 때문에 소괄호로 감싸서 표현식으로 만들어 준 것이다.

객체의 속성 중 함수를 축약해서 표현하는 방법은 아래와 같다.

```tsx
const obj = {
  onClickLog: function(e) {
    //...
  },
  onClickCompact(e) {
    //...
  }
}
```

단축 객체 표현의 반대는 객체 구조 분해이다.

```tsx
const props = obj.props;
const a = props.a;
const b = props.b;
```

위 코드는 아래와 같이 변경할 수 있다.

```tsx
const { props } = obj;
const { a, b } = props;
```

극단적으로는 아래와 같이도 줄일 수 있다.

```tsx
const { props: { a, b } } = obj;
```

이때 `a, b` 는 변수이지만 `props` 는 변수가 아니라는 것을 알고 있어야 한다.

구조 분해 문법에서는 기본 값을 지정할 수 있다. 아래는 if 문으로 기본 값을 지정하는 방식이다.

```tsx
type Foo = {
  foo: {
    a?: number;
  }
};
let pt: Foo = {
  foo: {}
}

let { a } = pt.foo;
if (a === undefined) {
  a = 0
}
console.log(a); // 0
```

대신 구조 분해 문법 내에서 기본 값을 지정할 수 있다.

```tsx
const { a = 0 } = pt.foo;
console.log(a); // 0

// 값이 있을 경우
let pt: Foo = {
  foo: {
		a: 1
	}
}
const { a = 0 } = pt.foo;
console.log(a); // 1
```

배열에도 구조 분해 문법을 사용할 수 있다.

```tsx
const point = [1, 2, 3];
const [x, y, z] = point;
const [, a, b] = point; // 첫 번째 요소 무시
```

함수 매개변수에도 구조 분해 문법을 사용할 수 있다.

```tsx
const point = [
  [1, 2, 3],
  [4, 5, 6],
];

point.forEach(([x, y, z]) => console.log(x + y + z));
```

단축 객체표현과 마찬가지로, 객체 구조 분해를 사용하면 **문법이 간결해지고, 변수를 사용할 때 실수를 줄일 수 있기에** 적극적으로 사용하는 것이 좋다.

## 함수 매개변수 기본 값 사용하기

자바스크립트에서 모든 매개변수는 생략이 가능하다. 매개변수를 지정하지 않는다면 `undefined` 값이 할당된다.

```jsx
function log2(a, b) {
  console.log(a, b); // undefined, undefined
}
log2();
```

옛날 자바스크립트에서 기본 값을 할당하고 싶을 때는 아래와 같이 했다.

```jsx
function parseNum(str, base) {
  base = base || 10;
  return parseInt(str, base);
}
```

하지만 모던 자바스크립트에서는 매개변수에 기본 값을 직접 지정할 수 있다.

```jsx
function parseNum(str, base = 10) {
  return parseInt(str, base);
}
```

매개변수에 기본 값을 직접 지정하면 코드가 간결해질 뿐만 아니라, base 가 선택적 매개변수라는 것을 명확히 나타내는 효과도 있다.

그리고 타입스크립트에서 기본 값을 기반으로 타입 추론을 하기 때문에, 마이그레이션 시 매개변수에 타입 구문을 넣지 않아도 된다.

## 저수준 Promise 나 Callback 대신 async/await 사용하기

async/await 를 사용하면 코드가 간결해져서 버그나 실수를 방지할 수 있고, 비동기 코드에 타입 정보가 전달되어 타입 추론이 가능하다.

```tsx
function getJson(url: string) {
  return fetch(url).then((response) => response.json);
}

function getJsonCallback(url: string, cb: (result: unknown) => void) {
  //...
}
```

```tsx
async function getJson(url: string) {
  const response = await fetch(url);
  return response.json();
}
```

## 연관 배열에 객체 대신 Map 과 Set 사용하기

인덱스 시그니처는 편리하지만 몇 가지 문제가 있다.

```tsx
function countWords(text: string) {
  const counts: {[word: string]: number} = {};
  for (const word of text.split(/[\s,.]+/)) {
    counts[word] = 1 + (counts[word] || 0);
  }
  return counts;
}
```

위 함수는 별다른 문제가 없어 보이지만, `"constructor"` 라는 특정 문자열이 들어왔을 때 문제가 발생한다.

```tsx
console.log(countWords('Objects have a constructor'));
/**
{
	Objects: 1,
	have: 1,
	a: 1,
	constructor: '1function Object() { [native code] }'
}
*/
```

`constructor` 의 초기 값은 `undefined` 가 아닌 `Object.prototype` 에 있는 **생성자 함수**이다. 원치 않는 값이며, 타입도 `number` 가 아닌 `string` 이다.

이런 문제를 방지하기 위해서는 Map 을 사용하는 것이 좋다.

```tsx
function countWords(text: string) {
  const counts = new Map<string, number>();
  for (const word of text.split(/[\s,.]+/)) {
    counts.set(word, 1 + (counts.get(word) || 0));
  }
  return counts;
}
```

## 타입스크립트에 use strict 넣지 않기

ES5 에서는 버그가 될 수 있는 버그 패턴에 오류를 표시해주는 엄격모드(“strict mode”) 가 있었다. 코드의 제일 처음에 `'use strict'` 를 넣으면 엄격모드가 활성화 된다.

```jsx
'use strict'
function foo() {
	x = 10; // Unresolved variable or type x
}
```

그러나 타입스크립트에서 수행되는 안전성 검사가 엄격 모드보다 훨씬 더 엄격한 체크를 하기 때문에 타입스크립트 코드에서 `'use strict'` 는 무의미하다.

실제로는 타입스크립트 컴파일러 생성하는 자바스크립트 코드에서 `'use strict'` 가 추가된다. 타입스크립트 컴파일러 옵션에 `alwaysStrict` 또는 `strict` 을 활성화 시키면 타입스크립트는 엄격 모드로 코드를 파싱하고 생성되는 자바스크립트에 `'use strict'` 를 추가한다.
