# 아이템47. 공개 API 에 등장하는 모든 타입을 Export 하기

타입스크립트를 사용하다보면 간혹 모듈에서 export 되지 않은 타입정보가 필요할 때가 있다. 만약 타입을 숨기고 싶어서 export 하지 않았다고 가정해보자

```tsx
interface SecretName {
  first: string;
  last: string;
}

interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  //...
}
```

위 코드는 getGift 함수만 export 할 수 있다. 그러나 타입들은 export 된 함수에 등장하기 때문에 추출이 가능하다.

```tsx
type MySanta = ReturnType<typeof getGift>;
type MyName = Parameters<typeof getGift>[0];
```

이렇게 공개 API 매개변수에 타입이 쓰여진다면 추출이 가능하다. 그러므로 굳이 공개 API 에 타입을 숨기려 하지 말고, **라이브러리 사용자를 위해 명시적으로 export 하는 것이 좋다**. 즉, 타입을 숨기려해도 알 수 있는 방법은 충분하다는 것이다.
