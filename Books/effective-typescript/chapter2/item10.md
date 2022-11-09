# 아이템10. 객체 래퍼 타입 피하기

자바스크립트에서는 객체를 제외하고도 7 가지의 타입(`string, number, boolean, null, undefined, symbol, bigint`)이 존재한다.

- 원시 래퍼 타입: `string, number, boolean, null, undefined, symbol, bigint`
- 래퍼 객체 타입: `String, Number, Boolean, Symbol`

예를 들면 기본형 string 의 경우 메서드를 가지고 있는 것처럼 보인다.

```tsx
let str: string = 'string';
str.charAt(3)
```

하지만 `charAt` 은 `string` 의 메서드가 아니다. `charAt` 은 `String` 래퍼 타입의 객체에 존재한다. string 을 사용할 때 자바스크립트에서는 내부적으로 많은 동작들이 일어난다. 자바스크립트는 원시래퍼타입인 기본형과 래퍼 타입을 자유롭게 변환한다.

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
