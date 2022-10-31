# 1장 - 타입스크립트 알아보기

타입스크립트는 독특한 언어이다. 인터프리터(Python, Ruby…)로 실행되는 것도 아니며, 저수준 언어(C, Java…)로 컴파일 되는 것도 아니다. 자바스크립트로 컴파일되며, 실행 역시 타입스크립트가 아닌 자바스크립트로 실행된다.

## 1. 타입스크립트와 자바스크립트 관계 이해하기

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

## 2. 타입스크립트 설정 이해하기

현재 타입스크립트 컴파일러는 매우 많은 설정(100개 이상)을 가지고 있다. 설정 파일을 사용하는 것이 좋다. 그렇게 해야 동료들이나 프레임워크가 타입스크립트를 어떻게 사용해야 할지 알 수가 있다. 타입스크립트는 설정에 따라 완전히 다른 언어라고 느낄 수 있다.

```tsx
// tsconfig.json
{
	"compilerOptions": {
		//...
	}
}
```

`noImplicitAny`  옵션은 변수들이 미리 정의된 타입을 가져야 하는지 여부를 체크한다.

```tsx
// type - a: any, b: any, add(): any
function add(a, b) {
  return a + b;
}
add(10, null);
```

이 코드는 `noImplicitAny` 옵션이 false 일 때 유효한 코드이다. any 타입을 지정한다면 타입 체커는 아무 것도 할 수가 없다. 위의 코드는 any 타입을 넣지 않았지만 any 타입으로 추론된다.

타입스크립트는 타입 정보를 가지고 있을 때 가장 효과적이기 때문에 `noImplicitAny: true` 로 설정해야 한다. 그렇게 된다면 타입스크립트가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상될 것이다.

`strictNullChecks` 옵션은 null 과 undefined 가 모든 타입에서 허용되는지 확인하는 설정이다.

```tsx
// strictNullChecks: false 일 때 정상
// strictNullChecks: true 일 때 오류 발생
const x: number = null;
```

만약 null 을 허용하고 싶다면 의도를 명시적으로 드러내어 오류 수정이 가능하다.

```tsx
const x: number | null = null;
```

`strictNullChecks` 옵션은 null 과 undefined 관련 오류를 잡아내는데 많은 도움이 되지만 코드 작성을 어렵게 한다. 프로젝트 초기에는 활성화 하는 것이 좋지만 아닌 경우에는 설정하지 않아도 된다. 프로젝트가 커지면 커질 수록 변경이 어렵기 때문에 **가급적 초반에 설정** 하는 것이 좋다.

이러한 모든 체크를 하고 싶다면 `strict` 설정을 하면 된다. 타입스크립트에서 `strict` 은 거의 모든 오류를 잡아낸다. 만약 같은 타입스크립트 코드이지만 다른 쪽에서 에러가 발생한다면 컴파일러 설정이 같은지 확인해야 한다.

## 3. 코드 생성과 타입이 관계 없음을 이해하기

크게 타입스크립트 컴파일러는 2 가지의 역할을 수행한다.

```
1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transfile) 한다.
2. 코드의 타입 오류를 체크한다.
```

여기서 1, 2번은 완전히 독립적이다. **타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 미치지 않는다**. 또한 자바스크립트 실행 시점에도 타입은 영향을 미치지 않는다.

### 타입 오류가 있는 코드도 컴파일이 가능하다

타입스크립트는 문제가 될만한 부분들을 알려주지만, 그렇다고 빌드를 멈추지는 않는다.

- 컴파일은 오직 코드 생성을 의미하기 때문에 코드에 오류가 있을 때는 “컴파일에 문제가 있다”라고 말하는 경우는 잘못된 것이다. 그러므로 “타입 체커에 문제가 있다”라고 하는 것이 더 정확한 표현이다.

만약 오류가 발생했을 때 컴파일을 하지 않기 위해서는 `noEmitOnError` 옵션을 활성화 하는 것이다.

### 런타임에는 타입 체크가 불가능하다

```tsx
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Rectangle | Square;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) { // Error
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

instanceof 는 런타임에 이뤄지지만 Rectangle 은 타입이기 때문에 런타임 시점에 아무것도 할 수가 없다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.

```tsx
function calculateArea(shape: Shape) {
  if ('height' in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

만약 이 문제를 해결하기 위해서는 shape 타입을 명확하게 해야 한다. **height 속성이 있는지 체크**를 해서 런타임에서도 타입 정보를 유지하도록 할 수 있다.

속성 체크는 런타임에 접근 가능한 값에만 관련있지만, 타입 체커도 `shape` 의 타입을 `Rectangle` 로 보정해주기 때문에 오류는 사라진다.

이 기법은 런타임에 타입 정보를 손 쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있다.

타입을 클래스로 만들어 오류를 해결할 수도 있다. 인터페이스는 타입으로만 사용가능하지만, 클래스를 타입으로 설정하면 *타입과 값으로 모두 사용*할 수 있다.

```tsx
class Square {
  width: number;
}

class Rectangle extends Square {
  height: number;
}

// 타입으로 참조
type Shape = Rectangle | Square;

function calculateArea(shape: Shape) {
	// 값으로 참조
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

### 타입 연산은 런타임에 영향을 주지 않는다

```tsx
function asNumber(val: number | string): number {
  return val as number;
}
```

위의 코드는 타입 체커에서 통과하지만 잘못된 방법을 사용한 것이다. `as number` 는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다.

값을 정제하기 위해서는 **런타임의 타입을 체크하고, 자바스크립트 연산을 통해 변환을 수행**해야 한다.

```tsx
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다

```tsx
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      // turn lightOn
      break;
    case false:
      // turn lightOff
      break;
    default:
      console.log('실행되지 않을까 걱정')
  }
}

interface LightApiResponse {
  lightSwitchValue: boolean;
}

async function setLight() {
  const response = await fetch('/light');
  const result: LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

`/light` 를 호출하면 그에 대한 결과로 `LightApiResponse` 를 응답 받을 것이라고 했지만 실제로는 `LightApiResponse` 의 `lightSwitchValue` 의 값이 문자열일 수도 있다.

이처럼 타입스크립트에서 런타임 타입과 선언된 타입이 맞지 않을 수도 있다. 선언된 타입은 언제든지 달라 질 수도 있다는 것을 명심해야 한다.

### 타입스크립트 타입으로 함수를 오버로드 할 수 없다

동일한 함수명에 매개변수만 다른 오버로드 기능을 타입스크립트에서는 지원하지 않는다. 타입과 런타임의 동작이 무관하기 때문에 함수 오버로딩을 불가능하다.

타입스크립트가 오버로딩을 지원하지만 온전히 타입 수준에서만 동작한다.

```tsx
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
  return a + b;
}

const three = add(1, 2);
const twelve = add('1', '2');
```

위의 두개의 선언문은 자바스크립트가 실행될 때 구현제만 남게 된다.

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않습니다

타입과 타입 연산자는 자바스크립트의 변환시점에 사라지기 때문에, 런타임 성능에는 아무런 영향을 미치지 않는다.

- 런타임 오버헤드가 없는 대신에 타입스크립트 컴파일러는 **빌드 타임 오버헤드가 존재**한다. 컴파일 시 오버헤드가 커질 경우 빌드 도구에서 타입 체크를 건너 뛸 수 있는 방법이 있다.
- 호환성과 성능사이의 선택은 컴파일 타깃과 언어 레벨의 문제이며 타입과는 무관하다.

## 4. 구조적 타이핑에 익숙해지기

자바스크립트는 [덕타이핑](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91)(객체의 변수 및 메소드의 집합이 객체의 타입을 결정하는 것) 기반이다.     
매개변수 값이 요구사항을 만족한다면 타업이 무엇인지 신경 쓰지 않는 동작을 그대로 모델링 한다.

```tsx
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NameVector {
  name: string;
  x: number;
  y: number;
}
const v: NameVector = { x: 3, y: 4, name: 'Zee' };
calculateLength(v); // 5
```

위 코드에서 `NameVector` 와 `Vector2D` 타입이 다른데 `calculateLength` 메서드가 문제 없이 실행된다. 그 이유는 `NameVector` 구조가 `Vector2D` 와 호환이 되기 때문에 `calculateLength` 메서드가 호출될 수 있는 것이다.

여기서 구조적 타이핑(structure typing) 이라는 용어가 사용된다. 
명시적 선언이나 이름을 기반으로 하는 **명목적 타입 시스템(Nominal Type System)** 인 Java, C#등과 다르고, 런타임에 타입을 체크하는 덕 타이핑과 다른 것이다.

```tsx
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function normalize(v: Vector3D) {
  const length = calculateLength(v); // 5
  console.log(length);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}

normalize({x: 3, y: 4, z: 5}) // x: 0.6, y: 0.8, z: 1
```

위의 코드에서 함수의 결과가 이상한 이유도 Vector3D 타입이 Vector2D 타입과 호환되기 때문에 오류가 발생하지 않았고, 타입 체커가 문제로 인식하지 않았다. 이런 경우를 오류 로처리하기 위한설정이 존재한다.

타입스크립트의 타입은 이처럼 확장에 열려있다. 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가지는 정확한 타입을 타입스크립트의 타입 시스템에서는 표현할 수 없다. 이러한 특정 때문에 가끔 당황스러운 결과가 발생하기도 한다.

```tsx
class C {
  foo: string;

  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' };
```

구조적 타이핑은 클래스와 관련된 할당문에서도 당황스러운 결과를 보인다.

```tsx
class C {
  foo: string | null;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' };
```

d 변수는 string 타입의 foo 속성을 가지고 있다. 게다가 생성자(Object.prototype으로부터 비롯된)를 가집니다. 따라서 구조적으로는 필요한 속성과 생성자를 가지고 있기 때문에 문제가 없는 것이다.

만약 Class C 에 함수나 연산로직이 추가되어 있다면 d 의 경우 문제가 발생합니다.

```tsx
class C {
  foo: string | null;
  constructor(foo: string) {
    this.foo = foo;
  }
  
  languageType(lang: string) {
    return typeof lang;
  }
}

const d: C = { foo: 'test' }; // Error - TS2741: Property 'languageType' is missing in type '{ foo: string; }' but required in type 'C'.
```

## 5. any 타입 지양하기

타입스크립트의 타입시스템은 **점진적이고 선택적**이다. 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입체커를 해제할 수 있기 때문에 선택적이다. 이 기능들의 핵심은 any 이다.

any를 사용하게 된다면 그 위험성을 알고 있어야 한다.

### any 타입에는 타입 안전성이 없다

```tsx
let age: number;
age = '12' as any;
age += 1;
console.log(age); // 121
console.log(typeof age); // string
```

age 가 number 타입으로 선언되었지만 as any 를 사용해서 string 타입을 할당했다. 런타임 시에는 문제가 발생하지 않는다. 그리고 타입 체커는 선언에 따라 number 타입으로 판단할 것이고 혼란스러운 상황이 발생한다.

### any는 함수 시그니처(번역: Contract)를 무시한다

호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입을 반환해야 하지만 `any` 를 사용하면 이런 약속을 어기는 상황이 발생한다.

```tsx
function calculateAge(birth: Date): number {
  return birth.getDate();
}

let birthDate: any = '1990-01-19';
calculateAge(birthDate);
```

위의 코드는 타입체커에서 문제로 인식하지 못한다. `calculateAge` 함수의 조건을 무시하게 된다. 이렇게 타입이 불일치하는 상황이 많아지면 다른 곳에서 문제를 일으키는 상황이 많이 발생하게 될 것이다.

### any 타입에는 언어 서비스가 적용되지 않는다

타입이 있다면 언어 서비스는 자동완성 기능을 제공해준다. 하지만 any 타입을 사용하면 아무런 도움도 받을 수 없다.

```tsx
interface Person {
  name: string;
  age: number;
}

const person1: Person = { name: 'choi', age: 11 }
const person2: any = { name: 'choi', age: 11 }
```

위의 코드에서 Person 타입에 name 을 firstName 으로 바꾸기 위해서 편집기의 Rename 기능을 사용하면 any 타입의 person2 는 아무런 기능도 제공받지 못하게 된다.

![Untitled 1](https://user-images.githubusercontent.com/63203480/199016144-7048061c-6905-42cf-b6c6-c11e86b3f1c1.png)

### any 타입은 코드 리팩터링 때 코드를 감춥니다

```tsx
interface ComponentProps {
  onSelectItem: (item: any) => void;
}

function renderSelector(props: ComponentProps): null {
  return null;
}

let selectedId: number = 1;
function handleSelectItem(item: any) {
  selectedId = item.id;
}

renderSelector({onSelectItem: handleSelectItem});
```

`item` 의 타입을 알 수 없는 상황이기에 `any` 타입으로 선언을 했다. 이 상황에서 `ComponentProps` 타입을 아래와 바꾸면 어떻게 될까?

```tsx
interface ComponentProps {
  onSelectItem: (id: number) => void;
}
```

`handleSelectItem` 메서드의 매개변수는 `any` 타입을 받고 있기에 문제가 발생하지 않는다. 만약 `any` 타입을 사용하지 않았다면 타입 체커가 문제를 발견했을 것이다

### any는 타입 설계를 감춰버립니다

객체를 정의할 때, 상태 객체의 설계를 감춰버리기 때문에 문제가 생긴다.

- 객체의 상태: 특정 시점에 객체가 가지고 있는 정보의 집합으로 객체의 구조적 특징

깔끔하고 정확하고 명료한 코드 작성을 위해 제대로된 타입설계는 필수이다. `any` 타입을 사용하면 타입의 설계가 불분명해진다. 협력하기 위해서는 설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋다.

### any 는 타입시스템의 신뢰도를 떨어트린다

사람은 항상 실수하기에 타입 체커가 실수를 잡아주고 코드의 신뢰도를 높여준다. any 타입을 사용하지 않는다면 **런타임에서 발견될 오류를 미리 잡고 신뢰도를 높일 수 있다**.

그리고 any 타입은 자바스크립트를 사용하는 것보다 더 힘든 상황을 만들 수 있다.

하지만 어쩔 수 없이 any 타입을 사용해야만 하는 경우도 존재한다.
