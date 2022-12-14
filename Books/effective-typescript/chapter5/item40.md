# 아이템40. 함수 안으로 타입 단언문 감추기

함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외상황까지 고려하면서 타입 정보를 힘들게 구성할 필요는 없다. 함수 내부에는 타입 단언을 사용하고, 함수 외부에 드러나는 타입을 정확히 명시하는 것으로 끝내는 것이 효율적이다. 프로젝트 전반적으로 타입 단언을 사용하는 것보다 **함수 안으로 타입 단언을 숨기는 것**이 더 좋은 설계이다.

어떤 함수가 자신의 마지막 호출을 캐시하는 코드입니다.

```tsx
declare function cacheLast<T extends Function>(fn: T): T;

declare function shallowEqual(a: any, b: any): boolean;

function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;

  return function(...args: any[]) { // Type '(...args: any[]) => any' is not assignable to type 'T'.
    if (!lastArgs || !shallowEqual(lastArgs, lastResult)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

타입스크립트는 반환문에 있는 함수와 원본 함수 `T` 타입이 어떤 관련이 있는지 알지 못하기 때문에 오류가 발생했다. 그러나 원본함수 T 타입과 동일한 매개변수로 호출되고, 반환 값도 예상과 같기 때문에 타입 단언문을 통해 오류를 제거하는 것은 문제가 되지 않는다.

```tsx
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;

  return function(...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, lastResult)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

함수 내부에 `any` 가 많이 있지만 타입 정의에는 `any` 가 없기 때문에 `cacheLast` 함수를 호출하는 쪽에서는 `any` 가 사용되었는지 알지 못한다.

타입 단언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야할 경우 함수안으로 숨겨서 사용하는 것이 좋다.
