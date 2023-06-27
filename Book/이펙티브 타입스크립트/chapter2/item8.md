# 아이템8. 타입 공간과 값 공간의 심벌(Symbol) 구분하기

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

`instanceof` 는 자바스크립트의 연산자기 때문에 `instanceof Cylinder`는 **타입을 참조하는 것이 아니라 함수를 참조**한다.

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
