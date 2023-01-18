# 아이템56. 정보를 감추려는 목적으로 private 사용하지 않기

타입스크립트에서 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수 있다.

```tsx
class Diary {
  private secret = 'secret';
}

const diary = new Diary();
diary.secret; // TS2341: Property 'secret' is private and only accessible within class 'Diary'.
```

그러나 public, protected, private 와 같은 접근제어자는 타입스크립트 키워드기 때문에 컴파일 이후에는 제거된다. 컴파일 이후에는 아래와 같이 변한다.

```jsx
class Diary {
  constructor() {
    this.secret = 'secret'
  }
}

const diary = new Diary();
diary.secret;
```

`private` 접근 제어자는 사라지고, `secret` 속성은 일반 속성이기 때문에 접근할 수 있다. 타입스크립트의 접근 제어자는 단지 컴파일 시점에 오류를 표시해 줄 뿐이고, 런타임에서는 아무런 효력이 없다. 심지어 단언문을 사용하면 타입스크립트에서도 접근이 가능하다.

```jsx
class Diary {
  private secret = 'secret';
}

const diary = new Diary();
(diary as any).secret;
```

즉, 클래스 내부 정보를 감추기 위해서 private 를 사용해서는 안된다는 것이다.

자바스크립트에서 **확실하게 정보를 숨기기 위해서는 클로저(closure)를 사용**하면 된다.

```tsx
declare function hash(text: string): number;

class PasswordChecker {
  checkPassword: (password: string) => boolean;

  constructor(passwordHash: number) {
    this.checkPassword = (password: string) => {
      return hash(password) === passwordHash;
    }
  }
} 
```

위 처럼 생성자에서 클로저를 만들어 낼 수 있다.

```tsx
const checker = new PasswordChecker(hash('s3cret'));
checker.checkPassword('s3cret');
```

`PasswordChecker` 클래스의 생성자 외부에서 `passwordHash` 변수에 접근이 불가하기 때문에 정보를 숨길 수 있다.

그런데 몇가지 주의사항이 있다. `passwordHash` 는 외부에서 접근할 수 없기 때문에 `passwordHash` 에 접근하기 위한 함수도 클래스 내부에 정의되어 있어야 한다. 그리고 **메서드 정의가 생성자 내부에 존재하게 되면, 인스턴스를 생성할 때마다 각 메서드의 복사본이 생성되기 때문에 메모리가 낭비**된다는 것을 기억해야 한다.

또 하나의 선택지로 비공개 필드는 #를 붙여서 타입 체크와 런타임 모두 비공개로 만들 수 있다.

```tsx
class Diary {
  #secret = 'secret';
}

const diary = new Diary();
diary.secret; // error
```

비공개 필드를 지원하지 않는 자바스크립트 버전으로 컴파일하게 될 경우, `WeakMap` 을 사용한 구현으로 대체된다.

이처럼 만약 설계 관점에서 캡슐화가 아닌 보안에 대해서 걱정하고 있다면 내장된 프로토타입과 함수에 대한 변조 문제를 알고 있어야 한다.
