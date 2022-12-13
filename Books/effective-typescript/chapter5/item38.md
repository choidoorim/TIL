# 아이템38. any 타입은 가능한 한 좁은 범위에서만 사용하기

전통적으로 프로그래밍 언어들의 타입 시스템은 완전히 정적이거나 완전히 동적으로 확실하게 분리되어 있다. 그러나 **타입스크립트의 타입 시스템은 선택적이고 점진적이기 때문에 정적과 동적의 특징을 모두** 갖는다.

프로그램의 일부분만 타입 시스템을 적용하는게 가능하기 때문에 Javascript 에서 Typescript 로 점진적인 마이그레이션이 가능하다. 마이그레이션을 할 때 코드 일부분에 타입 체크를 비활성화해주는 any 타입이 중요한 역할을 한다.

```tsx
function processBar(b: Bar) { //... }

function f() {
  const x = expressionReturningFoo();
  processBar(x); // Argument of type 'Foo' is not assignable to parameter of type 'Bar'. ts(2345)
}
```

`f()` 함수의 `x` 변수에 `Foo` 와 `Bar` 타입을 할당할 수 있다면 오류를 제거할 수 있는 방법은 2 가지이다.

```tsx
// X
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
}

// O
function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);
}
```

`f1()` 에서 사용된 방법보다 `f2()` 에서 사용된 방법이 더 좋다. 그 이유는 any 타입이 processBar 함수의 매개변수에만 사용된 표현식이기 때문에 다른 코드에는 영향을 미치지 않는다. `f1()` 에서 `x` 변수의 타입은 함수의 끝까지 `any` 타입인 반면에 `f2()` 는 함수호출 시에만 `any` 타입이다.

만약 `f1()` 이 `x` 를 반환하게 된다면 큰 문제가 발생한다.

```tsx
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x); 
  return x;
}

function g() {
  const foo = f1();
  foo.fooMethod();
}
```

`g` 함수에서 `f1` 함수를 호출해서 사용하고 있기 때문에 `any` 타입이 `foo` 변수의 타입에 영향을 미친다. 이렇게 함수에서 `**any` 를 반환하게 된다면 전염병처럼 그 영향력은 프로젝트 전반**에 퍼지게 된다. 반면에 f2 함수처럼 `any` 의 사용범위를 좁게 제한한다면 `any` 타입이 함수 밖으로 영향을 미치지 않는다.

코드에 `@ts-ignore` 를 사용하면 `any` 를 사용하지 않고 오류를 제거할 수 있다. 강제로 `any` 타입을 지정하기 보다는 `@ts-ignore` 를 사용하는 것이 더 낫다.

```tsx
function f2() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
}
```

하지만 근본적인 원인을 해결한 것이 아니기 때문에 다른 곳에서 더 큰 문제가 발생할 수도 있다. 타입 체커가 알려주는 오류는 다른 곳에서 큰 문제가 발생할 수 있기 때문에 원인을 찾아서 해결하는 것이 좋다.

아래는 객체 안에 속성타입에 오류가 발생하는 상황이다.

```tsx
interface Config {
  a: number;
  b: number;
  c: string;
}

const config: Config = {
  a: 1,
  b: 2,
  c: { // Type '{ foo: number; }' is not assignable to type 'string'.ts(2322)
    foo: 1,
  }
};
```

config 객체 전체를 `any` 로 선언하면 오류를 제거할 수 있다.

```tsx
const config: Config = {
  a: 1,
  b: 2,
  c: {
    foo: 1,
  }
} as any;
```

하지만 객체 전체를 `any` 로 선언하면 다른 속성들(a, b) 도 함께 타입체크가 되지 않게 된다. 따라서 **최소한의 범위에서만 `any` 를 사용**하는 것이 좋다.
