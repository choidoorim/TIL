# 아이템43. 몽키 패치보다는 안전한 타입을 사용하기

자바스크립트의 가장 유명한 특징 중 하나는, 객체와 클래스에 임의의 속성을 추가할 수 있을 만큼 유연하다는 것이다.

객체 속성을 추가하는 코드 스타일은 특히 JQuery 를 사용하는 코드에서 흔히볼 수 있다.

내장 기능의 프로토타입에도 속성을 추가할 수 있다.

```jsx
RegExp.prototype.monkey = 'Capuchin';
console.log(RegExp.prototype.monkey); // Capuchin
console.log(/123/.monkey); // Capuchin
```

하지만 이상한 결과를 보일 때가 있다. 정규식 `/123/` 에는 monkey 속성을 추가한 적이 없는데 `Capuchin` 값 이 들어있다. 객체에 임의의 속성을 추가하는 것은 좋지 않은 설계이다. 예를 들어 window 또는 DOM 노드에 데이터를 추가한다면 그 데이터는 기본적으로 전역 변수가 된다. 전역 변수를 사용하게 되면 **은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성**을 만들게 된다. 그렇게 된다면 함수 호출 시마다 Side Effect 를 고려하는 상황이 발생한다.

타입스크립트까지 적용된다면 또 다른 문제가 있다. 타입 체커는 `Document` 와 `HTMLElement` 의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.

```tsx
RegExp.prototype.monkey = 'Capuchin'; // TS2339: Property 'monkey' does not exist on type 'RegExp'
```

이 오류를 해결하기 위한 가장 간단한 방법은 any 단언문을 사용하는 것이다.

```tsx
(RegExp.prototype as any).monkey = 'Capuchin';
```

타입 체커에서 통과되지만 단점으로는 언어서비스를 사용하지 못하고, any 를 사용함으로써 타입 안전성을 상실하게 되는 것이다.

데이터를 분리하는 것이 좋은 해결책이다. 분리할 수 없는 경우 2가지의 차선책이 있다.

첫번째는 interface 의 특수기능 중 하나인 보강(Augmentation) 을 사용하는 것이다.

```tsx
interface Document {
  /** 몽키 패치 Genus 또는 Species **/
  monkey: string;
}
document.monkey = 'Tamarin'; // 정상
```

타입이 더 안전해지고, 속성에 주석이나 자동완성 기능을 사용할 수 있다. 그리고 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남기때문에 any 보다 더 나은 방법이다.

두번째로는 더 구체적인 타입 단언문을 사용하는 것이다.

```tsx
interface MonkeyDocument extends Document {
  monkey: string;
}
(document as MonkeyDocument).monkey = 'Macaque';
```

MonkeyDocument 는 Document 를 확장하기 때문에 타입 단언문은 정상이고 할당문의 타입은 안전하다. 그리고 Document 를 건드리지 않고 확장하기 때문에 모듈 영역 문제도 해결할 수 있다.

몽키패치를 남용해서는 안되며 더 잘 설계된 구조로 리팩터링하는 것이 좋다.
