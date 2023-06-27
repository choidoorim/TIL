# 아이템7. 타입이 값들의 집합이라고 생각하기

**“할당 가능한 값들의 집합”**을 타입이라고 생각하면 된다.

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
