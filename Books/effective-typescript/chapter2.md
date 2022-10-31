# 2장 - 타입스크립트의 타입 시스템

타입스크립트는 코드를 자바스크립트로 변환하는 역할도 하지만, 가장 중요한 역할은 타입시스템에 있다.

## 6. 편집기를 사용하여 타입시스템 탐색하기

타입스크립트를 설치하면 `"1. 타입스크립트 컴파일러(.tsc)` `2. 단독으로 실행할 수 있는 타입스크립트 서버"` 를 실행할 수 있다.

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

`& 연산자`는 두 타입의 교집합을 계산한다. 따라서 `Person` 과 `Lifespan` 타입 간의 공통으로 가지는 속성이 없기 때문에 `PersonSpan` 는 `never` 로 예상할 수 있다. 
하지만 `Person` 과 `Lifespan` 을 둘다 가지는 값은 인터섹션 타입에 속하게 된다.

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
interface Vector1D { 
	x: number; 
}
interface Vector2D extends VectorlD { 
	y: number; 
} 
interface Vector3D extends Vector2D {
 z: number; 
}
```

Vector3D는 Vector2D의 서브타입이고, Vector2D는 Vector1D의 서브타입 이다(클래스 관점 에서는 서브클래스).

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

string 을 상속(`<K extends string>`)한다는 의미는 string 의 부분집합 범위를 가지는 어떠한 타입이라는 뜻이다. string 리터널 타입, string 리터널 타입의 유니온 타입, string 자신을 포함하는 것이다.

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

