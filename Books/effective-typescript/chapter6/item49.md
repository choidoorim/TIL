# 아이템49. 콜백에서 this 에 대한 타입 제공하기

`let, const` 로 선언된 변수는 렉시컬 스코프인 반면에, `this` 는 다이나믹 스코프이다. 다이나믹 스코프는 정의된 방식이 아닌 호출된 방식에 따라서 달라진다.

this 는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 사용된다.

```jsx
class C {
  vals  = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val * val);
    }
  }
}

const c = new C();
c.logSquares(); // 1, 4, 9
```

`logSquares` 함수를 외부 변수에 넣고 호출하면 런타임에서 에러가 발생한다.

```jsx
class C {
  vals  = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val * val);
    }
  }
}

const c = new C();
const method = c.logSquares;
method(); // Cannot read properties of undefined (reading 'vals)
```

이유는 `c.logSquares()` 함수가 2 가지 작업을 수행하기 때문이다. `C.prototype.logSquares` 를 호출하고, `this` 값을 `c` 에 바인딩한다. 위 코드에서는 `logSquares` 의 참조변수를 이용해서 2 가지의 작업으로 분리했고, `this` 의 값은 `undefined` 로 설정된다.

Javascript 에서는 `this` 바인딩을 온전히 제어할 수 있는 방법이 있다. `call()` 을 사용하면 명시적으로 `this` 를 바인딩하여 문제를 해결할 수 있다.

```jsx
const c = new C();
const method = c.logSquares;
method.call(c);
```

`this` 바인딩은 종종 콜백함수에서 쓰인다.

```tsx
class ResetButton {
  constructor() {
    this.onClick = this.onClick.bind(this);
  }

  render() {
    return { text: 'Reset', onClick: this.onClick };
  }

  onClick() {
    alert(`Reset ${this}`);
  }
}

// ResetButton.prototype.onClick();
```

onClick 함수는 `ResetButton.prototype` 의 속성이다. 그렇기 때문에 ResetButton 의 모든 인스턴스에서 공유된다. 그러나 생성자에서 `this.onClick = …` 으로 바인딩을하면, `onClick` 속성에 `this` 가 바인딩되어 해당 인스턴스에 생성된다.

속성 탐색 순서(lookup sequence) 에서 onClick 인스턴스 속성(`this.onClick`)은 onClick 프로토타입(prototype) 속성보다 앞에 놓이므로, render() 메서드의 this.onClick 은 바인딩된 함수를 참조하게 된다.

```
자바스크립트는 프로토타입 기반 언어이며 클래스는 단지 간편 문법에 해당한다는 것을 기억하고 있어야 한다.
```

`onClick`을 Arrow Function 으로 바꾼다면 `ResetButton` 이 생성될 때마다 제대로 바인딩된 `this` 를 가지는 새 함수를 생성하게 된다.

```tsx
class ResetButton {
  render() {
    return { text: 'Reset', onClick: this.onClick };
  }

  onClick = () => {
    alert(`Reset ${this}`); // this 가 항상 인스턴스를 참조한다.
  }
}
```

자바스크립트로 트랜스파일된 코드는 아래와 같다.

```jsx
class ResetButton {
  constructor() {
    var _this = this;
    this.onClick = function () {
      alert("Reset " + _this);
    }
  }

  render() {
    return { text: 'Reset', onClick: this.onClick };
  }
}
```

`this` 바인딩은 자바스크립트의 동작이기 때문에, 타입스크립트 역시 `this` 바인딩을 그대로 모델링하게 된다.

만약 라이브러리 사용자가 콜백을 화살표 함수로 작성하고 this 를 참조하려고 한다면 타입스크립트가 문제를 잡아낸다.

```tsx
function addKeyListener(
  el: HTMLElement,
  fn:(this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn.call(el, e);
  });
}

class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, e => {
      this.innerHTML; // TS2339: Property 'innerHTML' does not exist on type 'Foo'
    })
  }
}
```

`this` 바인딩 문제는 콜백 함수의 매개변수에 this 를 추가하고, 콜백 함수를 `call` 로 호출해서 해결할 수 있다.

```tsx
function addKeyListener(
  el: HTMLElement,
  fn:(this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn.call(el, e);
  });
}
```

만약 `.call` 을 제거하고 fn 을 매개변수로 호출하면 타입 에러가 발생한다.

```tsx
function addKeyListener(
  el: HTMLElement,
  fn:(this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn.call(el, e);
  });
}

class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, e => {
      this.innerHTML; // TS2339: Property 'innerHTML' does not exist on type 'Foo'
    })
  }
}
```

따라서 콜백 함수에서 this 를 사용해야 한다면, 타입 정보를 명시해야 한다.
