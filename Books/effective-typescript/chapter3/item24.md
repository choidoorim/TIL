# 아이템24. 일관성 있는 별칭 사용하기

borough.location 배열에 loc 이라는 새로운 별칭을 만들었다.

```tsx
const borough = {
  name: 'Typescript',
  location: [40.688, -73.979],
}
const loc = borough.location;
loc[0] = 0;
console.log(borough); // { name: 'Typescript', location: [0, -73.979] }
```

별칭의 값을 변경하면 기존에 있던 원래 속성 값도 함께 변경된다. 별칭을 남발하면 제어 흐름을 분석하기 어렵다. 별칭은 신중하게 사용해서 코드를 이해하고, 오류도 쉽게 찾을 수 있도록 해야 한다.

아래는 다각형을 표현하는 자료구조를 표현한다.

```tsx
interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[];
  bbox?: BoundingBox;
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if (polygon.bbox) {
    if (pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] || pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1) {
      return false;
    }
  }
  //...
}
```

이 코드는 잘 동작하지만 `polygon.bbox` 가 반복되고 있다. 아래는 중복코드를 줄이기 위해서 변수를 뽑아낸 것이다.

```tsx
// TS2432 - Object is possibly 'undefined'.
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox; // type - box: BoundingBox | undefined
  if (polygon.bbox) {
    // type - box: BoundingBox | undefined
    // type - polygon.bbox: BoundingBox
    if (pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1]) {
      return false;
    }
  }
  //...
}
```

`strickNullChecks` 옵션을 활성화했다고 가정했을 때 코드는 동작하지만 편집기에서 에러가 발생한다. 왜냐하면 제어의 흐름을 방해했기 때문이다. `polygon.bbox` 의 타입을 `if` 문을 통해 좁혀 정제했지만 `box` 는 그렇지 못했기 때문이다. 이런 오류는 **“별칭은 일관성 있게 사용한다”** 라는 기본 원칙을 지키면 방지할 수 있다.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (box) { // ***
    if (pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1]) {
      return false;
    }
  }
  //...
}
```

타입 체커에 대한 문제는 해결했지만 읽는 사람에게는 `.bbox` 와 `box` 가 같은 값인데 다른 이름을 사용하기에 혼동을 줄 수 있다.

**객체 비구조화(Destructuring)를 사용**하면 간결한 문법으로 일관된 이름을 사용할 수 있다. 왜냐하면 구조분해 할당은 **속성을 로컬 변수로 만들어주기 때문**이다.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon; 
  if (bbox) {
    const { x, y } = bbox;
    if (pt.x < x[0] || pt.x > x[1] || pt.y < y[0] || pt.y > y[1]) {
      return false;
    }
  }
  //...
}
```

비구조화를 사용할 때는 2 가지를 주위해야 한다.

- `bbox` 의 `x, y` 가 선택적 속성일 경우에는 속성 체크를 부가적으로 더 해줘야 한다. 따라서 **타입의 경계에는 `null` 값을 추가**하는 것이 좋다.
- `bbox` 에는 선택적 속성이 적합했지만 `holes` 는 그렇지 못하다. `holes` 가 선택적이라면 값이 없거나 빈 배열이였을 것이다. **빈 배열을 통해 `holes` 값이 없음을 표현하는 것**이 좋다.

타입스크립트는 함수가 타입 정제를 무효화하지 않는다고 가정한다. 그러나 함수의 호출이 타입 정제를 무효화 할 수 있다는 것을 유의해야 한다.

`polygon.bbox` 로 사용하기보다는 `**bbox` 를 지역변수로 뽑아내서 사용**하면 `bbox` 타입은 정확하게 유지되지만 `polygon.bbox`의 값과 같게 유지되지 않을 수 있다. 즉, **속성보다는 지역 변수**를 통해 타입 정제를 믿을 수 있게 해야 한다.
