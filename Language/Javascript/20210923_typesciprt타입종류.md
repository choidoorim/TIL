## Typescript Type
타입 스크립트는 일반 변수, 매개 변수, 객체 속성 등에 타입을 지정할 수 있다.
- 형식
```typescript
function Example(a: Type, b: Type): RuturnType {
    return a + b;
}

let variable: Type = Example(1, 2);
```
- 예제
```typescript
function add (a: number, b: number): number {
    return a + b;
}

const sum_num: number = add(1, 2);
const err_sun_num: String = add(1, 2);   // ERROR
```
![1](https://user-images.githubusercontent.com/63203480/134499735-176b4d89-2fa3-4b1c-90c5-ef043a42ffde.png)

### Type 선언
#### Boolean
True/False 값을 나타내는 타입이다.
```typescript
let isDone: boolean = false;
```

#### Number (숫자)
모든 숫자를 지원하는 타입이다.   
부동 소수점은 number 타입이지만 Big Interger 는 bigint 타입이다.
```typescript
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
let big: bigint = 100n;
```

#### 문자열 (String)
문자열을 나타낸다. 큰 따옴표(") 또는 작은 따옴표(')를 사용하여 문자열 데이터를 묶는다.   
ES6 의 템플릿 문자열도 지원한다.
```typescript
let color: string = "blue";
color = 'red';
let myColor: string = `My color is ${color}`;
```

#### 배열 (Array)
순차적인 값을 갖는 일반 배열을 나타낸다.
배열은 2가지 방법으로 선언할 수 있다.
1. []
```typescript
let list: number[] = [1, 2, 3];
let str_list: string[] = ['a', 'b', 'c'];
```
2. Array<elemType>
```typescript
let list: Array<number> = [1, 2, 3];
let str_list: Array<string> = ['a', 'b', 'c'];
```

유니언 타입(다중 타입)의 문자열과 숫자를 동시에 가지는 배열도 선언할 수 있다.
```typescript
let array1: (string | number)[] = ["Apple", 1, 2, "Banana", "Mango", 3];
let array2: Array<string | number> = ["Apple", 1, 2, "Banana", "Mango", 3];
```
만약 배열의 값들의 타입들을 단언할 수 없을 경우 any 를 사용하면 된다.
```typescript
let someArr: any[] = [0, 1, {}, [], "str", false];
```
```readonly```, ```ReadonlyArray```타입을 사용해서 읽기 전용 배열도 생성가능하다.
```typescript
let arrA: readonly number[] = [1, 2, 3, 4];
let arrB: ReadonlyArray<number> = [2, 4, 6, 8];
arrA[0] = 123;      // ERROR
```
![1](https://user-images.githubusercontent.com/63203480/134507995-4f9ec102-ac14-4199-a054-7fa06651b5ca.png)

#### 튜플 (Tuple)
정해진 타입의 고정된 길이를 가지는 배열을 표현할 수 있다.
```typescript
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error

console.log(x[0].substring(1));  // OK

console.log(x[1].substring(1));  // ERROR
```
선언된 index 외 접근할 경우 오류가 발생한다.
```typescript
// ERROR
x[3] = "world";
console.log(x[5].toString());
```

#### 열거형 (Enum)
숫자 혹은 문자열 값 지합에 이름을 부여할 수 있는 타입으로, 
값의 종류가 일정한 범위로 정해져 있는 경우 유용하다.   
일반적으로 0부터 시작하며 1씩 증가한다.
```typescript
enum Color {
  Red,
  Green,
  Blue,
}
let c: Color = Color.Green;     // 2
```
수동으로 값을 변경할 수 있으며, 값을 변경한 부분부터 다시 1씩 증가한다.
```typescript
enum Color {
  Red = 20,
  Green,
  Blue,
}
let c: Color = Color.Green;     // 21
```
모든 값을 수동으로 설정할 수도 있다.
```typescript
enum Color {
  Red = 1,
  Green = 3,
  Blue = 4,
}
let c: Color = Color.Green;     // 3
```

#### any : 모든 타입을 허용
외부 라이브러리를 사용하여 타입을 단언할 수 없을 때 타입을 선언할 때 ```any```를 사용하여 타입 검사를 무시할 수 있다.
```typescript
let any: any = 111;
any = 'typescript';
any = {};
any = null;
```
만약 타입스크립트의 타입 시스템을 반드시 적용하기 위해서 ```any``` 사용을 금지하기 위해서는
컴파일 옵션에 ```"noImplicitAny": true``` 를 사용하면 ```any```사용시 ERROR 가 발생된다.

#### 알 수 없는 타입 (Unknown)
any 와 같은 최상위 타입으로 알 수 없는 타입을 의미한다.   
어떤 타입의 값도 할당할 수 있지만, ```unknown``` 을 다른 타입에 할당할 수는 없다.  
하지만 any 와 unknown 은 서로 할당할 수 있다.
```typescript
let u: unknown = 123;
let test1: number = u; //  ERROR
let test2: number = u as number; //타입을 선언하면 할당할 수 있다.
let test3: any = u;
```

#### Void
값을 반환하지 않는 함수에서 사용한다. 함수가 반환 타입을 명시하는 곳이다.
```typescript
function coding(msg: string): void {
  console.log(`Happy ${msg}`);
}
```

#### 객체 (Object)
typeof 연산자가 Object 로 반환하는 모든 타입을 나타낸다.   
컴파일러 옵션에서 ```"strict" = true``` 로 설정하면 null 은 포함하지 않는다.
```typescript
let obj: object = {};
let arr: object = [];
let func: object = function () {}; 
let date: object = new Date();
```
반복적인 사용을 할 경우, interface 나 type 을 사용할 수 있다.
```typescript
interface Users {
  name: string,
  age: number
}

let userA: Users = {
  name: 'juyoung',
  age: 27
};

let userB: Users = {
  name: 'jisu',
  age: 28,
  mail: true // Error
}
```

### Null and Undefined
모든 타입의 하위 타입으로, 여러 타입들에 할당할 수 있다.
```typescript
let num: number = undefined;
let str: string = null;
let obj: { a: 1, b: false } = undefined;
let arr: any[] = null;
let und: undefined = null;
let nul: null = undefined;
let voi: void = null;
```
컴파일 옵션에서 ```"strictNullChecks": true``` 를 통해 null 과 undefined 타입을 할당 할 수 없도록 하는 것이 가능하다.   
단, void 에는 undefined 할당이 가능하다.
```typescript
let vod: void = undefined;
```

### Never
절대 발생하지 않을 값을 나타내며 어떤 타입에도 적용할 수 없다. any 도 지정할 수 없다.   
항상 예외를 throw 하는 함수, error 를 return 하는 함수 등의 반환 타입이다.
```typescript
// Function returning never must not have a reachable end point
function error(message: string): never {
  throw new Error(message);
}
 
// Inferred return type is never
function fail() {
  return error("Something failed");
}
 
// Function returning never must not have a reachable end point
function infiniteLoop(): never {
  while (true) {}
}
```

### 유니언 (Union)
2개 이상의 타입을 허용하는 경우에 유니언이라고 한다.
```|``` 를 이용해 타입을 구분한다.
```typescript
let union: (string | number);
union = 'Hello World';
union = 777;
```