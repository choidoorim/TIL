# 아이템18. 매핑된 타입을 사용하여 값을 동기화 시키기

새로운 속성이 추가된 것을 알려주는 기능이 있다.

```tsx
interface ScatterProps {
  xs: number[];
  ys: number[];

  xRange: [number, number];
  yRange: [number, number];
  color: string;

  onClick: (x: number, y: number, index: number) => void;
}

const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
}

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      if (k !== 'onClick') {
        return true;
      }
    }
  }
  return false;
}
```

매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적이다. 위 예제처럼 매핑된 타입을 사용해 타입스크립트가 코드에 제약을 강제하도록 할 수도 있다.

매핑된 타입을 통해서 관련된 값과 타입을 동기화 시킬 수 있다. 그리고 **인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려**해야 한다.
