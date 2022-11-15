# 아이템22. 타입 좁히기

타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 뜻한다.

- (1~7): 타입을 좁히는 방법

일반적으로 null 체크**(1)**가 있다.

```tsx
const el = document.getElementById('foo'); // HTMLElement | null
if (el) {
	// 타입이 HTMLElement
  el.innerHTML = '';
} else {
  alert('No Element #foo');
}
```

첫 번째 if 문 블록에서 `HTMLElement | null` 타입의 `null` 을 제거하므로 더 좁은 타입이 되어 작업이 훨씬 쉬워진다.

타입 체커는 일반적으로 조건문의 타입 좁히기는 잘하지만, 타입 별칭이 있다면 그렇지 못할 수도 있다.

분기문에서 예외를 던지거나 함수를 반환해서 타입을 좁힐 수도 있다**(2)**.

```tsx
const el = document.getElementById('foo');
if (!el) {
  throw new Error();
}
el.innerHTML = ''; // 타입이 HTMLElement
```

또한 instanceof 를 사용해서도 타입을 좁힐 수 있다**(3)**.

```tsx
function contains(text: string, search: string|RegExp) {
  if (search instanceof RegExp) {
    // 타입이 RegExp
  }
  // 타입이 string
}
```

속성체크로도 타입을 좁힐 수 있고**(4)**, Array.isArray 와 같은 내장함수로도 타입을 좁힐 수 있다**(5)**.

이렇게 타입스크립트는 일반적으로 조건문을 통한 타입좁히기에는 능하다. 그러나 타입을 섣불리 판단하는 실수를 저지르기 쉽기 때문에 잘 확인해야 한다.

```tsx
const el = document.getElementById('foo');
if (typeof el === 'object') {
  el; // 타입이 여전히 HTMLElement | null
}
```

위 코드는 잘못된 방식으로 타입 좁히기를 하고 있다. **자바스크립트에서는 `null` 의 타입이 `object`**이기 때문에 if 조건절에서 `null` 이 제거되지 못했다.

또한 기본형의 값이 잘못되어도 비슷한 사례가 발생한다.

```tsx
function foo(x?: number|string|null) {
  if (!x) {
    x; // 타입이 string | number | null | undefined
  }
}
```

위 코드에서 **빈문자열(’’), 0 모두 false** 이기 때문에 타입은 전혀 좁혀지지 못하고 `x` 변수의 타입은 블록내에 여전히 `string` 또는 `number` 가 된다.

타입을 좁히는 또 다른 방법은 명시적 태그를 붙히는 것이다**(6)**.

```tsx
interface UploadEvent { type: 'upload', filename: string; contents: string }
interface DownloadEvent { type: 'download', filename: string; }
type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
  switch (e.type) {
    case "upload":
      e; // 타입이 UploadEvent
      break;
    case "download":
      e; // 타입이 DownloadEvent
      break;
  }
}
```

이러한 패턴은 **태그된 유니온(tagged union)** 또는 **구별된 유니온(discriminated union)** 이라고 불리며 타입스크립트에서 자주 볼 수 있다.

만약 타입스크립트가 타입을 식별하고 있지 못하다면, 식별을 위해서 커스텀 함수를 도입해도 된다**(7)**.

```tsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // 타입이 HTMLInputElement
    return el.value;
  }
  el; // 타입이 HTMLElement
  return el.textContent;
}
```

이러한 기법을 **사용자 정의 타입 가드**라고 한다. `el is HTMLInputElement` 는 함수의 반환이 true 인 경우 타입 체커에게 매개변수 타입을 좁힐 수 있다고 알려준다.

타입 가드를 통해서 배열과 객체의 타입 좁히기를 할 수 있다. 예를 들어, 배열에서 어떤 탐색을 수행할 때 undefined 가 될 수 있는 타입을 사용할 수 있다.

```tsx
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
); // [undefined, 'Michael'];
```

filter 함수를 통해 undefined 를 걷어낼라고 해도 잘 되지 않을 것이다. members 의 타입은 여전히 `(*string* | *undefined*)[]` 이다.

바로 이런 경우 타입 가드를 통해 해결할 수 있다.

```tsx
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}

const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(isDefined); // const members: string[]
```
