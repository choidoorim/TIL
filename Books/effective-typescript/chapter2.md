# 2장 - 타입스크립트의 타입 시스템

타입스크립트는 코드를 자바스크립트로 변환하는 역할도 하지만, 가장 중요한 역할은 타입시스템에 있다.

## 6. 편집기를 사용하여 타입시스템 탐색하기

타입스크립트를 설치하면 “ `1. 타입스크립트 컴파일러(.tsc)` `2. 단독으로 실행할 수 있는 타입스크립트 서버` ” 를 실행할 수 있다.

보통은 타입스크립트 컴파일러를 실행하는 것이 목적이지만, 타입스크립트 서버 또한 언어서비스를 제공한다는 점에서 중요하다. 언어서비스에서는 코드 자동완성, 명세, 검사, 검색, 리팩토링이 포함된다.

만약 타입스크립트에서 추론한 타입이 기대한 것과 다르다면 타입을 직접 명시하고, 문제가 발생하는 부분을 찾아야 한다.

## 7. 타입이 값들의 집합이라고 생각하기

**할당 가능한 값들의 집합**을 타입이라고 생각하면 된다.

가장 작은 집합은 **아무것도 포함하지 않는 공집합**이며, 타입스크립트에서는 `never` 타입이다.

```tsx
const num: never = 1;
// ERROR - TS2322: Type 'number' is not assignable to type 'never'.
```

never 타입으로 선언된 변수의 범위는 공집합이기 때문에 아무 값도 할당할 수 없다.

그 다음으로 작은 집합은 **한가지 값만 포함하는 타입**이다. 타입스크립트에서 유닛이라고 불리는 리터널 타입이다.

```tsx
type A = 'A';
type B = 'B';
type Twelve = 12;

const num: Twelve = 11; 
// ERROR - TS2322: Type '11' is not assignable to type '12'.
```

2, 3개로 묶기 위해서는 유니온 타입을 사용해야 한다.

```tsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;

const word: AB12 = 13;
// ERROR - TS2322: Type '13' is not assignable to type 'AB12'.
```

유니온 타입은 값 집합들의 합집합을 뜻한다.

집합의 관점에서 타입체커는 **하나의 집합이 다른 집합의 부분 집합인지 검사하는 것**이라고 볼 수 있다.

```tsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
const ab: AB = Math.random() < 0.5 ? 'A' : 'B'; // 정상
const ab12: AB12 = ab; // 정상
```

구조적 타이핑 규칙들은 **어떠한 값이 다른 속성도 가질 수 있음을 의미**한다. 심지어 함수 호출의 매개변수에서도 다른 속성을 가질 수 있습니다.

```tsx
interface Person {
  name: string;
	age: number;
}

interface Lifespan {
	name: string;
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;
/**
 * PersonSpan
 * {
 *   name: string;
 *   age: number;
 *   birth: Date;
 *   death?: Date;
 * }
 */

const ps: PersonSpan = {
  name: 'name',
  birth: new Date(),
  death: new Date(),
}
```

`& 연산자`는 두 타입의 교집합을 계산한다. 따라서 `Person` 과 `Lifespan` 타입 간의 공통으로 가지는 속성이 없기 때문에 `PersonSpan` 는 `never` 로 예상할 수 있다. 하지만 `Person` 과 `Lifespan` 을 둘 다 가지는 값은 인터섹션 타입에 속하게 된다.

- 인터섹션 타입: 여러 타입을 모두 만족하는 하나의 타입을 의미

조금 더 일반적으로 `PersonSpan` 타입을 지정하는 방법은 `extends` 키워드를 사용하는 것이다.

```tsx
interface Person {
  name: string;
}

interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

서브 타입이란 어떤 집합이 다른 집합의 부분 집합이라는 것이다.

```tsx
interface VectorlD { 
	x: number; 
}
interface Vector2D extends VectorlD { 
	y: number; 
} 
interface Vector3D extends Vector2D {
 z: number; 
}
```

Vector3D는 Vector2D의 서브타입이고, Vector2D는 VectorlD의 서브타입 이다(클래스 관점 에서는 서브클래스).

- 서브클래스: 상속 받는 클래스

`extends` 는 제네릭 타입에서 한정자(어떤 대상 값을 **불가시키기 위해** 사용하는 것)로도 쓰이며, “~의 부분집합” 을 의미하기도 한다.

```tsx
function getKey<K extends string>(val: any, key: K) {
	//...
}

getKey({}, 'x');
getKey({}, Math.random() < 0.5 ? 'a' : 'b');
getKey({}, 12); // ERROR - TS2345: Argument of type 'number' is not assignable to parameter of type 'string'.
```

string을 상속(`<K extends string>`)한다는 의미는 string 의 부분집합 범위를 가지는 어떠한 타입이라는 뜻이다. string 리터널 타입, string 리터널 타입의 유니온 타입, string 자신을 포함하는 것이다.

위의 코드에서 `getKey({}, 12);` 오류 중 ‘할당할 수 없습니다’ 는 ‘상속할 수 없습니다’ 로 바꿀 수 있다.

```tsx
// ERROR
const list1: number[] = [1, 2];
const tuple1: [number, number] = list1; //ERROR - S2322: Type 'number[]' is not assignable to type '[number, number]'. Target requires 2 element(s) but source may have fewer.

// OK
const tuple2: [number, number] = [1, 2];
const list2: number[] = tuple2;
```

위의 코드에서 `number[]` 는 `[number, number]` 의 부분집합이 아니기 때문에 할당할 수 없다. 그러나 반대로 할당하면 동작된다.

요약하자면, “A는 B를 상속”,  “A는 B에 할당 가능”, “A는 B의 서브타입” 은 “A는 B의 부분 집

합” 과 같은의미이다. 예를 들면, `<K extends string>` 에서 k 는 string 을 상속, k 는 string 에 할당 가능, K 는 string 의 서브타입, K 는 string 의 부분집합 이라는 것이다.

```tsx
/**
 * <K extends string>
 * 1. K 는 string 을 상속
 * 2. K 는 string 에 할당 가능
 * 3. K 는 string 의 서브타입
 * 4. K 는 string 의 부분집합
 */

// 타입 A 는 string 의 부분집합/서브타입 이기 때문에 string 타입의 변수에 type A 를 할당할 수 있는 것이다
type A = 'A';
const a: A = 'A';

const c: string = a;
```

## 8. 타입 공간과 값 공간의 심벌(Symbol) 구분하기

심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있다.

```tsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius // ERROR - TS2339: Property 'radius' does not exist on type '{}'
  }
}
```

Cylinder 는 타입(`interface Cylinder`)으로도 쓰이고 값(`const Cylinder`)으로도 쓰이고 있다.

`instanceof` 는 자바스크립트의 연산자기 때문에 `instanceof Cylinder` 는 타입을 참조하는 것이 아니라 함수를 참조한다.

자바스크립트에서는 객체 내의 각 속성을 로컬 변수로 만들어주는 **구조분해(Destructuring) 할당**을 사용할 수 있다.

```tsx
/**
 * ERROR
 * TS7031: Binding element 'Person' implicitly has an 'any' type.
 * TS7031: Binding element 'string' implicitly has an 'any' type.
 * TS2300: Duplicate identifier 'string'.
 */
function email({ person: Person, subject: string, body: string, }) {
  //...
}
```

하지만 타입스크립트에서 구조분해를 사용하려면 이상한 오류가 발생한다. Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려고 했기 때문입니다.

```tsx
function email({ person, subject, body, }: { person: Person, subject: string, body: string }) {
  //...
}
```

타입과 값을 구분하면 문제를 해결할 수 있다.

- 자바스크립트의 타입은 string, number, boolean, undefined, object, function 의 6개 런타임 타입만 존재한다

## 9. 타입 단언보다는 타입 선언을 사용하기

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

## 10. 객체 래퍼 타입 피하기

자바스크립트에서는 객체를 제외하고도 7 가지의 타입(`string, number, boolean, null, undefined, symbol, bigint`)이 존재한다.

- 원시 래퍼 타입: `string, number, boolean, null, undefined, symbol, bigint`
- 래퍼 객체 타입: `String, Number, Boolean, Symbol`

예를 들면 기본형 string 의 경우 메서드를 가지고 있는 것처럼 보인다.

```tsx
let str: string = 'string';
str.charAt(3)
```

하지만 `charAt` 은 `string` 의 메서드가 아니다. `charAt` 은 `String` 래퍼 타입 객체에 존재한다. string 을 사용할 때 자바스크립트에서는 내부적으로 많은 동작들이 일어난다. 자바스크립트는 원시래퍼타입인 기본형과 래퍼 타입을 자유롭게 변환한다.

예를 들면 `string` 기본형에 `charAt` 같은 메서드를 사용할 때, 자바스크립트는 **기본형을 `String` 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다**.

`String.prototype` 을 몽키 패치한다면 내부 동작을 확인할 수 있다.

```tsx
// 몽키패치: 런타임의 프로그램 기능을 수정해서 사용하는 기법, 자바스크립트에서는 주로 프로토타입을 수정
const originalCharAt = String.prototype.charAt;

String.prototype.charAt = function(pos) {
  console.log(this, typeof this, pos);
  return originalCharAt.call(this, pos); 
};
console.log('primitive' .charAt(3)); // m
```

하지만 `string` 과 `String` 은 항상 동일하게 동작하지 않는다.

```tsx
typeof "hello" // string
typeof new String("hello") // object

console.log("hello" === new String("hello")) // false
console.log(new String("hello") === new String("hello")) // false 
```

또한 객체 래퍼 타입은 당황스러운 동작을 보여질 때도 있다. 예를 들어 어떤 속성이 기본형에 할당된다면 사라진다.

```jsx
let x = 'hello';
x.language = 'English';
console.log(x.language); // undefined

let y = new String('hello');
y.language = 'English';
console.log(y.language); // English
```

그 이유는 x 가 `String` 객체로 변환된 후 `language` 속성이 추가되었고, `**language` 속성이 추가된 객체는 버려진 것**이다. String 객체를 직접 생성해서 사용한다면 사라지지 않는다. 왜냐하면 **기본형을 객체로 래핑하고 버리는 과정을 하지 않기 때문**이다.

null 과 undefined 타입을 제외하면 모든 타입에 객체 래퍼 타입이 존재한다. 이러한 래퍼 타입 덕분에 기본형 값에 메서드를 사용할 수 있는 것이다.

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다. 그런데 string 을 사용할 때 주의할 점이 있다.

```tsx
function getStringLen(foo: String) {
  return foo.length;
}

console.log(getStringLen("hello")) // 5
console.log(getStringLen(new String("hello typescript"))) // 16

function isGreeting(phrase: String) {
  return ['hello', 'good day'].includes(phrase); // TS2345: Argument of type 'String' is not assignable to parameter of type 'string'.   'string' is a primitive, but 'String' is a wrapper object. Prefer using 'string' when possible.
}
```

string 을 매개변수로 받는 메서드에 String 객체를 전달하는 순간 문제가 발생한다.

```tsx
function isGreeting(phrase: String) {
  return ['hello', 'good day'].includes(phrase); // TS2345: Argument of type 'String' is not assignable to parameter of type 'string'.   'string' is a primitive, but 'String' is a wrapper object. Prefer using 'string' when possible.
}

const hello: String = "hello";
function sayHello(word: string) {
  return `Say ${word}`
}
console.log(sayHello(hello)) // TS2345: Argument of type 'String' is not assignable to parameter of type 'string'.   'string' is a primitive, but 'String' is a wrapper object. Prefer using 'string' when possible.
```

왜냐하면 `**string` 은 `String` 에 할당할 수 있지만, `String` 은 `string` 에 할당할 수 없기 때문**이다.

그렇기에 일반적으로 **기본형 타입을 사용**하는 것이 좋다.

그러나 `**new` 키워드 없이 `BigInt` 나 `Symbol` 을 호출하는 경우**는 기본형을 생성하기 때문에 괜찮다.

```tsx
const hello1 = new String('hello')
const hello2 = String('hello')

console.log(typeof hello1); // object
console.log(typeof hello2); // string
```

## 11. 잉여 속성 체크의 한계 인지하기

타입이 선언된 변수에 객체 리터널을 할당할 때, 타입스크립트는 **해당 타입의 속성이 있는지**, 그리고 **그 외의 속성은 없는지**를 확인한다.

```tsx
// EX1
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const room: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // TS2322: Type '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' is not assignable to type 'Room'.   Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'.
}
```

구조적 타이핑 관점에서는 오류가 발생하지 않아야 한다. 반대로 room 타입의 변수에 임시 변수를 도입해보면 문제가 없는 것을 확인할 수 있다.

```tsx
// EX2
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
}

const room: Room = obj;
```

obj 타입은 `{numDoors: number, ceilingHeightFt: number, elephant: string}` 로 추론되기에 obj 타입은 Room 타입의 부분 집합을 포함하므로, Room 타입에 할당가능하며 타입체커에서도 통과된다.

`EX1`에서는 구조적 타이핑의 중요한 종류의 오류를 잡을 수 있도록 **잉여 속성 체크라는 과정이 추가**되었다. 그러나 잉여 속성 체크도 조건에 따라 동작하지 않는다는 한계가 있고, 할당 가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수도 있다. **잉여 속성 체크와 할당 가능 검사는 별도**라는 것을 기억하고 있어야 한다.

타입스크립트는 런타임에 오류를 표시하는 것뿐만이 아니라, 의도와 다르게 작성된 코드까지 찾으려 한다.

```tsx
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    // ...
  }
  // ...
}

createWindow({
  title: 'Spider Solitaire',
  darkmode: true, // TS2345: Argument of type '{ title: string; darkmode: boolean; }' is not assignable to parameter of type 'Options'.   Object literal may only specify known properties, but 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?
})
```

하지만 `Options` 타입의 범위는 상당히 넓다. darkMode 속성에 `boolean` 타입이 아닌 다른 값이 지정된 경우를 제외하면, `**string` 타입의 title 속성과 또 다른 속성을 가지고 있는 모든 객체는 `Options` 타입범위에 속한다**.

```tsx
interface Options {
  title: string;
  darkMode?: boolean;
}

const o1: Options = document;
const o2: Options = new HTMLAnchorElement;
```

`document`와 `HTMLAnchorElement`의 인스턴스 모두 string 타입인 title 속성을 가지고 있기 때문에 할당문(`o1, o2`)은 정상이 된다.

**잉여 속성 체크를 사용한다면 기본적으로 타입 시스템의 구조적인 본질을 해치지 않고, 객체 리터널에 알 수 없는 속성을 허용하지 않으면서 위의 문제점들을 방지**할 수 있다.

```tsx
const o1: Options = { darkmode: true, title: 'Sko Free' }; // 오류
// TS2322: Type '{ darkmode: boolean; title: string; }' is not assignable to type 'Options'.   Object literal may only specify known properties, but 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?

const intermediate = { darkmode: true, title: 'Sko Free' }; // 정상
const o2: Options = intermediate;
```

변수에 할당되는 값이 첫번째 케이스는 **객체 리터널이기에 오류가 발생**하지만, 두번째 케이스는 **객체 리터널이 아니기 때문에 잉여 속성 체크가 적용되지 않고 오류가 발생하지 않는다**.

[타입 스크립트에서는 신선도(Freshness)](https://radlohead.gitbook.io/typescript-deep-dive/type-system/freshness)라는 개념을 제공한다. 모든 객체 리터널은 초기에 `fresh` 하다고 간주되며, 1) 타입 단언(type assertion)을 하거나, 2) 타입 추론에 의해 object literal의 타입이 확장되면 `freshness`가 사라지게 된다. 특정 변수에 객체 리터널을 할당할 경우 2 가지중 한가지가 발생하게 되므로 `freshness` 가 사라지게 되고, **함수에 인자로 객체 리터널을 할당할 경우 `fresh` 한 상태로 전달**되게 된다.

```tsx
// 출처: https://toss.tech/article/typescript-type-compatibility
type Food = {
  protein: number;
  carbohydrates: number;
  fat: number;
}

type Burger = Food & {
  burgerBrand: string;
}

function calculateCalorie(food: Food){
  return food.protein * 4
    + food.carbohydrates * 4
    + food.fat * 9
}

const burger = {
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
}

const calorie = calculateCalorie(burger)

const calorie2 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
})
```

잉여 속성 체크는 타입 단언문을 사용할 때도 적용되지 않는다.

```tsx
const o: Options = { darkmode: true, title: 'Sko Free' } as Options; // 정상
```

이것이 명백하게 단언문보다 선언문을 사용해야 되는 이유이다.

약한(weak) 타입에서도 비슷한 체크가 작동한다.

```tsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true };
const op: LineChartOptions = opts; // TS2559: Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.
```

구조적 관점에서 LineChartOptions 타입은 모든 속성이 선택적인 약한 타입이므로 **타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행**한다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타같은 실수를 바로 잡는데 효과적이다.

선택적 필드를 포함하는 Options 와 같은 타입에는 유용하지만, 적용범위도 제한적이고 오직 객체 리터널에만 적용된다.

## 12. 함수 표현식에 타입 적용하기

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

## 13. 타입과 인터페이스의 차이점 알기

타입스크립트에서 Named Type(명명된 타입)은 2 가지가 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}

interface IState {
  name: string;
  capital: string;
}
```

클래스를 사용해서도 가능하지만 클래스는 값으로도 쓰일 수 있는 자바스크립트 런타임의 개념이다.

대부분의 경우에는 타입과 인터페이스를 선택해서 사용해도 된다. 하지만 두 차이를 알고, 같은 상황에서는 동일한 방법으로 명명된 타입(Named Type) 을 정의해서 일관성을 유지해야 한다.

- **참고**: 타입에 I(인터페이스), T(타입) 라는 접두사를 붙이는 것은 현재 지양해야 되는 스타일로 여겨진다. 예제에서만 사용!

### 비슷한점

**1. 인덱스 시그니처**

인덱스 시그니처는 타입과 인터페이스 모두 사용할 수 있다.

```tsx
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

**2. 함수 타입 정의**

또한 함수 타입도 정의가 가능하다.

```tsx
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = x => '' + x;
const toStrI: IFn = x => '' + x;
```

자바스크립트에서 함수는 호출가능한 객체이기 때문에 아래와 같은 코드도 가능하다.

```tsx
type TFnWithProperties = {
  (x: number): number;
  prop: string;
}

interface IFnWithProperties {
  (x: number): number;
  prop: string;
}
```

**3. 제너릭 타입**

타입과 인터페이스 모두 제너릭이 가능하다.

```tsx
type TPair<T> = {
  first: T,
  second: T;
}

interface IPair<T> {
  first: T,
  second: T;
} 
```

**4. 확장**

인터페이스는 타입을 확장할 수 있고, 타입은 인터페이스를 확장할 수 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}

interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop  = IState & { population: number };
```

주의 해야할 점은 **인터페이스는 유니온 타입과 같은 복잡한 타입을 확장하지 못한다는 것**이다.

복잡한 타입의 확장을 위해서는 타입(`type`)과 `&` 를 사용해야 한다.

**5 . 클래스 구현(`implements`)**

클래스 구현은 타입과 인터페이스 모두 사용할 수 있다.

```tsx
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}

class StateT implements TState {
  name: string = '';
  capital: string = '';
}
class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

### 다른 점

**1. 유니온 타입**

유니온 타입은 있지만 유니온 인터페이스는 없다.

```tsx
type AorB = 'a' | 'b';
```

인터페이스는 타입을 확장할 수는 있지만 유니온은 할 수 없다.

별도의 2개 이상의 타입을 하나의 변수로 매핑하는 인터페이스를 만들 수 있다.

```tsx
type Input = { /* ... */ };
type Output = { /* ... */ };

interface VariableMap {
  [name: string]: Input | Output;
}
```

또는 유니온 타입에 속성을 추가한 타입을 만들 수도 있다.

```tsx
type NamedVariable = (Input | Output) & { name: string };
```

이러한 타입은 인터페이스로 구현할 수 없다. 보통 타입은 인터페이스보다 쓰임새가 다양하다.

튜블과 배열 타입도 type 키워드를 통해 더 간결하게 표현할 수 있다.

```tsx
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
const namedNums: NamedNums = ['asd', 1, 2, 3];
```

인터페이스로도 튜플과 비슷하게 구현할 수 있기는 하다.

```tsx
interface Tuple {
  0: number;
  1: number;
  length: 2;
}

const t: Tuple = [10, 20];
```

하지만 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드를 사용할 수 없다. 그러므로 **튜플은 타입으로 구현**하는게 더 좋다.

**2. 확장,보강(Augment)하는 방법**

인터페이스가 가지고 있는 타입에 없는 몇 가지 기능 중 하나는, 보강 기법을 사용할 수 있다는 것이다.

```tsx
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000,
}
```

위의 예제처럼 속성을 확장하는 것을 **선언 병합(declaration merging)** 이라고 한다. 이처럼 선언병합을 사용하기 위해서는 반드시 인터페이스를 사용해야 한다.

타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합한다.

예로, Array 인터페이스는 `lib.es5.d.ts` 에 선언된 인터페이스가 사용된다. 그러나 `tsconfig.json`의 lib 목록에 ES2015를 추가하면 타입스크립트는 `lib.es2015.d.ts`에 선언된 인터페이스를 병합한다. 이처럼 병합을 통해 다른 Array 인터페이스에 추가된다.

타입은 기존타입에 추가적인 보강이 없는 경우에만 사용해야 한다.

### 결론

**복잡한 타입을 가져야 한다면 무조건 타입 별칭(Type Alias) 를 사용**해야 한다.

그러나 타입과 인터페이스 2 가지로 표현이 가능하다면 일관성과 보강의 관점에서 봐야 한다. 인터페이스를 사용하는 코드베이스일 경우 인터페이스를 사용하고, 타입을 사용하는 코드베이스일 경우 타입을 사용하면 된다.

어떤 **API에 대한 타입을 작성해야 한다면 인터페이스**가 좋다. API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문이다.

그러나 **프로젝트 내부적으로 사용되는 타입에는 선언병합이 발생할 수 있기 때문에 이럴 때는 타입을 사용**하는 것이 좋다.

## 14. 타입 연산과 제너릭 사용으로 반복 줄이기

중복된 코드를 열심히 제거하면서 DRY 원칙을 지켰던 개발자라도 타입에 대해서는 간과했을 수 있다.

타입 중복 문제는 코드 중복 만큼 많은 문제를 발생시킨다. 타입의 중복을 줄일 수 있는 가장 간단한 방법은 이름을 붙이는 것이다.

```tsx
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {
  /* ... */ 
}
```

예를 들어 몇몇 함수가 같은 타입 시그니처를 공유하고 있다고 가정해보자. 그러면 해당 시그니처를 **명명된 타입(Named Type)으로 분리**해 낼 수 있다.

```tsx
function get(url: string, opts: Options): Promise<Response> { /* ... */ }
function post(url: string, opts: Options): Promise<Response> { /* ... */ }

type HTTPFunction = (url: string, opts:Options): Promise<Response>;
const get1: HTTPFunction = (url, opts) => {};
const post1: HTTPFunction = (url, opts) => {};
```

그리고 한 인터페이스가 다른 인터페이스를 **확장**해서 반복을 줄일 수 있다.

```tsx
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
type PersonWithBirthDate = Person & { birth: Date }
```

예를 들어, 전체 어플리케이션의 상태를 표현하는 `State` 타입과 부분을 표현하는 `TopNavState` 가 있는 경우가 있다고 해보자.

```tsx
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

이런 경우에는 TopNavState 를 확장하기 보다는 State 의 부분집합으로 TopNavState 를 정의하는 것이 더 좋은 방법이다.

```tsx
// X
interface State extends TopNavState {
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}

// O
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```

이러한 방법은 전체 어플리케이션 상태를 하나의 인터페이스로 유지할 수 있게 해준다. `State` 를 인덱싱해서 속성의 타입에서 중복을 제거할 수 있다. `State` 의 `pageTitle` 의 타입이 바껴도 `TopNavState` 에 반영이 된다.

**매핑된 타입**을 사용하면 반복되는 코드를 더 줄일 수 있다.

```tsx
type TopNavState = {
  [k in 'userId'| 'pageTitle' | 'recentFiles']: State[k];
}
```

매핑된 타입은 배열의 필드를 반복문 도는 것과 같은 방식이다. 이 패턴은 표준 라이브러리에서도 사용할 수 있으며  `Pick` 이라고 한다. `Pick` 은 제네릭 타입으로 `T, K` 2 가지 타입을 받아서 결과를 return 한다.

```tsx
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

태그된 유니온에서도 다른 형태의 중복이 발생한다.

```tsx
interface SaveAction {
  type: 'save',
}

interface LoadAction {
  type: 'load',
}
type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load'; // 중복
```

Action 유니온을 인덱싱하면 타입의 반복 없이 `ActionType` 을 정의할 수 있다.

```tsx
type ActionType = Action['type'];
```

Action 유니온에 타입을 더 추가하면 ActionType 은 자동적으로 그 타입을 포함하게 된다.

```tsx
//...
interface TestAction {
  type: 'test',
}

type ActionType = Action['type']; // "load" | "save" | "test"
```

보통 update 기능은 create 에서 사용한 매개변수와 동일하며, 타입 대부분이 선택적 필드가 된다.

```tsx
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
```

이런 경우 매핑된 타입과 `keyof` 를 사용하면 `Options` 로부터 `OptionsUpdate` 를 간편하게 만들 수 있다.

```tsx
type OptionsUpdate = {
  [k in keyof Options]?: Options[k];
}
```

이러한 패턴 역시 일반적으로 사용되며 표준 라이브러리에 Partial 라는 기능으로 제공되고 있다.

```tsx
type OptionsUpdate = Partial<Options>;
```
