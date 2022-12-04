# 아이템32. 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유니온 타입의 속성을 가지는 인터페이스를 작성중이라면 인터페이스의 유니온 타입을 사용하는게 더 알맞지 않은가 검토해봐야 한다.

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FilePaint | LinePaint | PointPaint;
}
```

layout 속성은 모양이 그려지는 방법과 위치(둥근 모서리, 직선)를 제어하고, paint 속성은 스타일(파란선, 굵은선, 점선 등) 을 제어한다.

layout 타입이 `LineLayout` 이면서 paint 속성이 `FillPaint` 타입인 것은 말이되지 않는다.

더 나은 방법으로는 모델링하려면 **각각 타입의 계층을 분리된 인터페이스**로 만들어야 한다.

```tsx
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

이런 방식으로 유효한 타입만 정의하는 방식으로 Layer 를 정의한다면 잘못된 조합으로 섞이는 것을 방지할 수 있다.

이러한 패턴의 가장 일반적인 예시는 Tagged Union(태그된 유니온)이다.

- **Tagged Union**: 특정 속성 값을 통해 값이 속한 브랜치를 식별할 수 있는 방법

```tsx
interface Layer {
  type: 'fill', 'line', 'point';
  layout: FillLayout | LineLayout | PointLayout;
  paint: FilePaint | LinePaint | PointPaint;
}
```

예를 들면 `type: ‘fill’` 과 `LineLayout, PointPaint` 타입이 같이 쓰이는 것은 말이 되지 않는다. 이런 경우를 대비해 인터페이스 유니온으로 바꾸는 것이 좋다.

```tsx
interface FillLayer {
  type: 'fill';
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: 'layout';
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

type 속성은 태그이며 런타임에서 어떤 타입의 Layer 가 사용되는지 판단하는데 사용된다. 타입스크립트는 type 태그를 사용해서 Layer 타입을 좁힐 수 있다.

```tsx
function drawLayer(layer: Layer) {
  if (layer.type === 'fill') {
    const { paint, layout } = layer; // Type - FillPaint, FillLayout
  } else if (layer.type === 'line') {
    const { paint, layout } = layer; // Type - LinePaint, LineLayout
  } else if (layer.type === 'layout') {
    const { paint, layout } = layer; // Type - PointPaint, PointLayout
  }
}
```

타입스크립트가 코드의 정확성을 체크하는데 도움이 되지만, 타입 분기 후 layer 가 포함된 동일한 코드가 분기되는 것은 어수선하다.

아래 처럼 타입 코드처럼 주석에 타입 정보를 담는 것은 좋지 못하다.

```tsx
interface Person {
  name: string;
	// 다음은 동시에 둘다 있거나 없습니다
  placeOfBirth?: string;
  dateOfBirth?: string;
}
```

그렇기에 두 개의 속성을 하나로 모으는 것이 더 나은 설계이다.

```tsx
interface Person {
  name: string;
  birth?: {
    place: string;
    date: string;
  }
}

const alanT: Person = {
  name: 'Alan T',
  birth: { // Property 'date' is missing in type '{ place: string; }' but required in type '{ place: string; date: string; }'.ts(2741)
    place: 'London',
  }
}
```

이제는 `place` 만 있고 `date` 가 없는 경우에는 오류가 발생한다. Person 객체를 받는 함수는 birth 만 체크하면 된다.

```tsx
function eulogize(p: Person) {
  console.log(p.name);
  const { birth } = p;
  if (birth) {
    const { place, date } = birth;
  }
}
```

만약 타입의 구조를 손 댈 수 없는 상황일 경우에는 인터페이스 유니온을 사용해서 속성 사이의 관계를 모델링 할 수 있다.

```tsx
interface Name {
  name: string;
}
interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}
type Person = Name | PersonWithBirth;

function eulogize(p: Person) {
  if ('placeOfBirth' in p) {
    p;
    const { dateOfBirth } = p;
  }
}
```

중첩된 객체에서도 속성간의 관계를 더 명확하게 하는데 도움을 준다.

이렇게 유니온 타입의 속성을 여러 개 갖는 인터페이스에서는 속성 간의 관계가 명확하지 않기 때문에 실수가 자주 일어나므로 주의해야 한다. 또한 타입스크립트가 제어 흐름을 이해하기 위해 타입에 태그를 넣는 것을 고려해야 한다.
