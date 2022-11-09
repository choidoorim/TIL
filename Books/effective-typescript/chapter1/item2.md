# 2. 타입스크립트 설정 이해하기

현재 타입스크립트 컴파일러는 매우 많은 설정(100개 이상)을 가지고 있다. 설정 파일을 사용하는 것이 좋다. 그렇게 해야 동료들이나 프레임워크가 타입스크립트를 어떻게 사용해야 할지 알 수가 있다. 타입스크립트는 설정에 따라 완전히 다른 언어라고 느낄 수 있다.

```tsx
// tsconfig.json
{
	"compilerOptions": {
		//...
	}
}
```

`noImplicitAny`  옵션은 변수들이 미리 정의된 타입을 가져야 하는지 여부를 체크한다.

```tsx
// type - a: any, b: any, add(): any
function add(a, b) {
  return a + b;
}
add(10, null);
```

이 코드는 `noImplicitAny` 옵션이 false 일 때 유효한 코드이다. any 타입을 지정한다면 타입 체커는 아무 것도 할 수가 없다. 위의 코드는 any 타입을 넣지 않았지만 any 타입으로 추론된다.

타입스크립트는 타입 정보를 가지고 있을 때 가장 효과적이기 때문에 `noImplicitAny: true` 로 설정해야 한다. 그렇게 된다면 타입스크립트가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상될 것이다.

`strictNullChecks` 옵션은 null 과 undefined 가 모든 타입에서 허용되는지 확인하는 설정이다.

```tsx
// strictNullChecks: false 일 때 정상
// strictNullChecks: true 일 때 오류 발생
const x: number = null;
```

만약 null 을 허용하고 싶다면 의도를 명시적으로 드러내어 오류 수정이 가능하다.

```tsx
const x: number | null = null;
```

`strictNullChecks` 옵션은 null 과 undefined 관련 오류를 잡아내는데 많은 도움이 되지만 코드 작성을 어렵게 한다. 프로젝트 초기에는 활성화 하는 것이 좋지만 아닌 경우에는 설정하지 않아도 된다. 프로젝트가 커지면 커질 수록 변경이 어렵기 때문에 **가급적 초반에 설정** 하는 것이 좋다.

이러한 모든 체크를 하고 싶다면 `strict` 설정을 하면 된다. 타입스크립트에서 `strict` 은 거의 모든 오류를 잡아낸다. 만약 같은 타입스크립트 코드이지만 다른 쪽에서 에러가 발생한다면 컴파일러 설정이 같은지 확인해야 한다.
