# 아이템35. 데이터가 아닌, API 와 명세를 보고 타입 만들기

파일 형식, API, 명세(specification) 등 우리가 다루는 타입 중 몇 가지는 외부에서 비롯된 것이다. 이러한 경우 타입을 직접 작성하지 않고 자동으로 생성할 수 있다.

명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있도록 도와준다. Feature의 경계 상자를 계산하는 함수를 작성한다고 하자.

```tsx
function calculateBoundingBox(feature: Feature): BoundingBox | null {
    let box: BoundingBox | null = null;

    const helper = (coords: any[]) => {
      //...
    };

    const { geometry } = feature;
    if (geometry) {
      helper(geometry.coordinates);
    }

    return box;
}
```

`Feature` 타입은 명시적으로 정의된 적이 없다. `Feature` 타입이 `@types/geojson` 에 있기 때문에 공식 GeoJSON 명세를 사용하는 것이 좋다. 하지만 `GeoJSON` 의 `Feature` 타입을 넣으면 타입스크립트에서 오류가 발생한다.

```tsx
import {Feature} from 'geojson';

function calculateBoundingBox(feature: Feature): BoundingBox | null {
    let box: BoundingBox | null = null;

    const helper = (coords: any[]) => {
      //...
    };

    const { geometry } = feature;
    if (geometry) {
      helper(geometry.coordinates); 
			// Property 'coordinates' does not exist on type 'Geometry'.
			// Property 'coordinates' does not exist on type 'GeometryCollection<Geometry>'.
    }

    return box;
}
```

```tsx
// geojson
/**
 * MultiPolygon geometry object.
 * https://tools.ietf.org/html/rfc7946#section-3.1.7
 */
export interface MultiPolygon extends GeoJsonObject {
    type: 'MultiPolygon';
    coordinates: Position[][][];
}

/**
 * Geometry Collection
 * https://tools.ietf.org/html/rfc7946#section-3.1.8
 */
export interface GeometryCollection<G extends Geometry = Geometry> extends GeoJsonObject {
    type: 'GeometryCollection';
    geometries: G[];
}
```

`geometry` 에 `coordinates` 가 있을거라고 가정한게 문제가 됐다. `geojson` 라이브러리를 확인해보면 다른 도형 타입과 다르게 `GeometryCollection` 에는 `coordinates` 가 없다. 이 오류를 고치기 위해서는 `GeometryCollection` 을 명시적으로 차단해야 한다.

```tsx
import {Feature} from 'geojson';

function calculateBoundingBox(feature: Feature): BoundingBox | null {
    let box: BoundingBox | null = null;

    const helper = (coords: any[]) => {
      //...
    };

    const { geometry } = feature;
    if (geometry) {
      if (geometry.type === 'GeometryCollection') {
        throw new Error();
      }
      helper(geometry.coordinates);
    }

    return box;
}
```

타입스크립트에서는 타입을 체크하는 방법으로 도형을 정제할 수 있기 때문에 정제된 타입에 한해서 `geometry.coordinates` 를 허용한다.

기존에는 GeoJSON 에 대한 타입을 직접 작성했을 수 있다. 직접 작성한 타입에는 `GeometryCollection` 과 같은 예외가 포함되지 않았을 것이고, 완벽하기 힘들었을 것이다. 하지만 명세를 기반으로 작성한다면 현재까지 경험한 데이터뿐만 아니라 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있다.

이처럼 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 작성하는 것이 좋다.
