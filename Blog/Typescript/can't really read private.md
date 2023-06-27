# Typescript - Private 는 진짜로 읽을 수 없을까?

아래 `private` 으로 선언된 필드들을 가지고 있는 클래스가 있다.
```typescript
class Foo {
  private num: number;
  private str: string;

  constructor(num: number, str: string) {
    this.num = num;
    this.str = str;
  }
}

const foo = new Foo(1, 'str');
```

만약 num 이나 str 필드를 읽으려고 한다면 에러가 발생할 것이다. 
```typescript
// TS2341: Property 'num' is private and only accessible within class 'Foo'.
foo.num;
```
에러 메시지를 확인해보면 num 속성은 private 이고, 클래스 내부에서만 접근이 가능하다는 것이다.      
그렇다면 private 필드를 진짜 읽을 수 없을까?    

```typescript
const num = foo.num; // 타입체크 시 에러를 발생
console.log(num); // But. 런타임 시 1 을 출력한다
// or
const num = foo["num"]; // 타입체크 시 에러를 발생하지 않는다. 에러를 피하기 위한 방법
console.log(num); // But. 런타임 시 1 을 출력한다
```

위의 코드 결과를 확인해보면 `1` 값이 출력되는 것을 확인할 수 있다. 즉, 정상적으로 실행된다는 것이다. 어떻게 이런 일이 가능할까?
이 상황을 이해하기 위해서는 타입스크립트를 이해해야 한다.    

타입스크립트 컴파일러는 2 가지의 역할을 수행합니다.

```text
1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transfile) 한다.
2. 코드의 타입 오류를 체크한다.
```

여기서 중요한 점은 1번과 2번이 완전히 독립적이라는 것이다. 코드의 타입 오류에 문제가 발생해도 트랜스파일 과정을 수행합니다. 
이것 때문에 위의 `Foo` 클래스 예제와 같은 상황이 발생하게 되는 것입니다.    
Typescript 는 Javascript 코드로 트랜스파일하는 것과 타입 오류를 런타임 전에 체크해주기 때문에 **런타임에는 타입 체크가 불가능**합니다.        
즉, 위 예제에서 `private` 필드의 값을 읽을 수 있던 이유는 **`private` 이 Javascript 에는 존재하지 않고 Typescript 에만 존재**하기 때문입니다.     
실제로 Typescript 코드가 Javascript 로 트랜스파일되면 아래와 같다고 생각하면 될 것 같습니다.

```javascript
class Foo {
  num;
  str;
  constructor(num, str) {
    this.num = num;
    this.str = str;
  }
}
```

그렇다면 Javascript Class 에는 private 가 없을까? 아닙니다. Javascript 에서도 존재합니다.    
바로 [hash(#)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes/Private_class_fields) 를 통해 class 에 private 필드를 선언하여 사용할 수 있습니다.

```javascript
class Foo {
  #num;
  #str;
  constructor(num, str) {
    this.#num = num;
    this.#str = str;
  }
}

const foo = new Foo(1, 'str');
console.log(foo.#num); // SyntaxError
```
Typescript 의 private 와 다르게 ECMAScript 의 hash(#) private 를 사용하면 필드 값에 접근하는 것을 완벽하게 보호할 수 있다.

```text
- Typescript - soft privacy
- Javascript - hard privacy
```

그렇다면 왜 Typescript 에서는 hash(#) private 를 사용하지 않을까요?     

### 1. Typescript 의 지원 범위
Typescript 는 현재 Target 이 ES6(ECMAScript 2015) 이상이 아닐 경우 hash(#) private 를 지원할 수 없다고 합니다.    
하지만 **Typescript 의 private 는 모든 Target 에서 동작**합니다.

### 2. 속도
Typescript 의 private 은 다른 일반 필드/속성과 다르지 않기 때문에 런타임 환경에서 속도차이가 없습니다.    
하지만 hash(#) private 필드는 WeakMaps 를 사용하기 때문에 속도가 느릴 수도 있다고 합니다.

- [WeakMap](https://www.zerocho.com/category/ECMAScript/post/576cad515eb04d4c1aa35077) : 키가 Object 인 Map

## 결론
Typescript 를 사용하면서 Javascript 에도 존재하는 class private 기능을 왜 사용하지 않는지 궁금해서 알아보고 정리하게 되었습니다.    
만약 Class 내 필드에 접근하는 것을 완벽히 막고 싶다면 hash(#) private 를 사용하면 될 것으로 생각된다. 
하지만 대부분의 경우에는 Typescript 의 private 를 사용해서 컴파일 시에 코드 상에서 충분히 해결이 가능할 것이라고 생각이 든다.    
결국에는 개인의 상황에 알맞게 사용하면 될 것 같다는 생각이 듭니다.

## 참고
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html  

https://www.zerocho.com/category/ECMAScript/post/576cad515eb04d4c1aa35077
