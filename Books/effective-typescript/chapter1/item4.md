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
