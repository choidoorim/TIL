# 아이템12. 함수 표현식에 타입 적용하기

자바스크립트와 타입스크립트에서는 함수 문장(statement)과 함수 표현식(expression)을 다르게 인식한다.

```tsx
function rollDice1(sides: number): number { /* ... */ }           // 문장
const rollDice2 = function(sides: number): number { /* ... */ }   // 표현식
const rollDice3 = (sides: number):number => { /* ... */ };        // 표현식
```

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있기 때문이다.

```tsx
type RollDice = (sides: number) => number;

// TS2322: Type '(sides: string) => number' is not assignable to type 'RollDice'.   Types of parameters 'sides' and 'sides' are incompatible.     Type 'number' is not assignable to type 'string'.
const rollDice2: RollDice = function(sides: string): number { /* ... */ } // Error
const rollDice3: RollDice = sides => { 
  return 1;
}; // Ok
```

sides 인자에 마우스를 올려놓고 확인해보면 이미 `number` 타입으로 추론하고 있는 것을 확인할 수 있다.

### 장점 1

불필요한 타입의 선언을 줄여준다.

```tsx
function add(a: number, b: number) {
  return a + b;
}
function sub(a: number, b: number) {
  return a - b;
}
function mul(a: number, b: number) {
  return a * b;
}
function div(a: number, b: number) {
  return a / b;
}

// 타입 적용
type BinaryFn = (a: number, b: number) => number;
const add1: BinaryFn = (a, b) => a + b
const sub1: BinaryFn = (a, b) => a - b
const mul1: BinaryFn = (a, b) => a * b
const div1: BinaryFn = (a, b) => a / b
```

위처럼 반복되는 함수 형식을 하나의 함수 타입으로 통합할 수도 있다. 또한 타입 구문이 적어지고, 로직이 분명하게 드러나는 장점이 있다.

```tsx
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if(!response.ok) {
    return new Error();
  }
  return response;
}
//------------------------
const checkedFetch: typeof fetch = async (input, init) => {
	// TS2322: Type '(input: RequestInfo | URL, init: RequestInit) => Promise<Error | Response>' is not assignable to type '(input: RequestInfo | URL, init?: RequestInit) => Promise<Response>'.
	// Type 'Promise<Error | Response>' is not assignable to type 'Promise<Response>'.
	// Type 'Error | Response' is not assignable to type 'Response'.
	// Type 'Error' is missing the following properties from type 'Response': headers, ok, redirected, status, and 11 more.
  const response = await fetch(input, init);
  if(!response.ok) {
    return new Error(); // 문제점
  } 
  return response;
}
```

만약 함수 문장을 표현식으로 바꾸고 타입을 지정해주면 인자(`input, init`) 의 타입을 추론할 수 있고, 함수(`checkedFetch`) 의 반환타입을 보장하게 된다. 예를 들어 위처럼 throw 를 사용하지 않고 return 을 사용하게 되면 타입스크립트가 실수를 잡아낸다.

함수의 매개변수에 타입을 선언하기보다, **함수 표현식 전체 타입을 정의하는 것이 코드도 간결해지고 안전**하다.

- 다른 함수의 시그니처를 사용하려면 `typeof fn` 을 사용하면 된다.
