## 3. 코드 생성과 타입이 관계 없음을 이해하기

크게 타입스크립트 컴파일러는 2 가지의 역할을 수행한다.

```
1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transfile) 한다.
2. 코드의 타입 오류를 체크한다.
```

여기서 1, 2번은 완전히 독립적이다. **타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 미치지 않는다**. 또한 자바스크립트 실행 시점에도 타입은 영향을 미치지 않는다.

### 타입 오류가 있는 코드도 컴파일이 가능하다

타입스크립트는 문제가 될만한 부분들을 알려주지만, 그렇다고 빌드를 멈추지는 않는다.

- 컴파일은 오직 코드 생성을 의미하기 때문에 코드에 오류가 있을 때는 “컴파일에 문제가 있다”라고 말하는 경우는 잘못된 것이다. 그러므로 “타입 체커에 문제가 있다”라고 하는 것이 더 정확한 표현이다.

만약 오류가 발생했을 때 컴파일을 하지 않기 위해서는 `noEmitOnError` 옵션을 활성화 하는 것이다.

### 런타임에는 타입 체크가 불가능하다

```tsx
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Rectangle | Square;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) { // Error
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

instanceof 는 런타임에 이뤄지지만 Rectangle 은 타입이기 때문에 런타임 시점에 아무것도 할 수가 없다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.

```tsx
function calculateArea(shape: Shape) {
  if ('height' in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

만약 이 문제를 해결하기 위해서는 shape 타입을 명확하게 해야 한다. **height 속성이 있는지 체크**를 해서 런타임에서도 타입 정보를 유지하도록 할 수 있다.

속성 체크는 런타임에 접근 가능한 값에만 관련있지만, 타입 체커도 `shape` 의 타입을 `Rectangle` 로 보정해주기 때문에 오류는 사라진다.

이 기법은 런타임에 타입 정보를 손 쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있다.

타입을 클래스로 만들어 오류를 해결할 수도 있다. 인터페이스는 타입으로만 사용가능하지만, 클래스를 타입으로 설정하면 *타입과 값으로 모두 사용*할 수 있다.

```tsx
class Square {
  width: number;
}

class Rectangle extends Square {
  height: number;
}

// 타입으로 참조
type Shape = Rectangle | Square;

function calculateArea(shape: Shape) {
	// 값으로 참조
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

### 타입 연산은 런타임에 영향을 주지 않는다

```tsx
function asNumber(val: number | string): number {
  return val as number;
}
```

위의 코드는 타입 체커에서 통과하지만 잘못된 방법을 사용한 것이다. `as number` 는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다.

값을 정제하기 위해서는 **런타임의 타입을 체크하고, 자바스크립트 연산을 통해 변환을 수행**해야 한다.

```tsx
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다

```tsx
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      // turn lightOn
      break;
    case false:
      // turn lightOff
      break;
    default:
      console.log('실행되지 않을까 걱정')
  }
}

interface LightApiResponse {
  lightSwitchValue: boolean;
}

async function setLight() {
  const response = await fetch('/light');
  const result: LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

`/light` 를 호출하면 그에 대한 결과로 `LightApiResponse` 를 응답 받을 것이라고 했지만 실제로는 `LightApiResponse` 의 `lightSwitchValue` 의 값이 문자열일 수도 있다.

이처럼 타입스크립트에서 런타임 타입과 선언된 타입이 맞지 않을 수도 있다. 선언된 타입은 언제든지 달라 질 수도 있다는 것을 명심해야 한다.

### 타입스크립트 타입으로 함수를 오버로드 할 수 없다

동일한 함수명에 매개변수만 다른 오버로드 기능을 타입스크립트에서는 지원하지 않는다. 타입과 런타임의 동작이 무관하기 때문에 함수 오버로딩을 불가능하다.

타입스크립트가 오버로딩을 지원하지만 온전히 타입 수준에서만 동작한다.

```tsx
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
  return a + b;
}

const three = add(1, 2);
const twelve = add('1', '2');
```

위의 두개의 선언문은 자바스크립트가 실행될 때 구현제만 남게 된다.

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않습니다

타입과 타입 연산자는 자바스크립트의 변환시점에 사라지기 때문에, 런타임 성능에는 아무런 영향을 미치지 않는다.

- 런타임 오버헤드가 없는 대신에 타입스크립트 컴파일러는 **빌드 타임 오버헤드가 존재**한다. 컴파일 시 오버헤드가 커질 경우 빌드 도구에서 타입 체크를 건너 뛸 수 있는 방법이 있다.
- 호환성과 성능사이의 선택은 컴파일 타깃과 언어 레벨의 문제이며 타입과는 무관하다.
