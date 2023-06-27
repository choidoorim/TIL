# 아이템29. 사용할 때는 너그럽게, 생성할 때는 엄격하게

함수의 매개변수 타입은 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 한다.

매개변수의 타입이 넒으면 사용하기 편리하지만, 반환 타입의 범위가 넓으면 불편하다. 즉, 사용하기 편한 API 일수록 반환 타입이 엄격해야 한다.

아래 예제는 3D 매핑 API로 카메라의 위치를 지정하고 경계 박스의 뷰포트를 계산하는 방법을 제공한다. 일부 값은 건드리지 않으면서 동시에 다른 값을 설정할 수 있어야하기 때문에  `CameraOptions` 타입의 필드는 모두 선택적이어야 한다. `viewportForBounds` 함수는 `LngLatBounds` 라는 자유로운 타입을 매개변수로 받는다.

`LngLat` 는 3 개의 타입을 받을 수 있기 때문에 `LngLatBounds` 가 가능한 형태는 19 가지로 매우 자유롭다.

```tsx
type LngLat =
  | { lng: number; lat: number }
  | { lon: number; lat: number }
  | [number, number];

type LngLatBounds =
  | { northeast: LngLat; southwest: LngLat }
  | [LngLat, LngLat]
  | [number, number, number, number];

interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

function focusOnFeature() {
	const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const { center: { lat, lng }, zoom } = camera;
  // TS2339: Property 'lat' does not exist on type 'LngLat'.
  // TS2339: Property 'lng' does not exist on type 'LngLat'.
  zoom; // type: number or undefined
}
```

위 예제의 오류는 `lat`, `lng` 속성이 없고 `zoom` 속성만 있기에 발생한 문제이다.

수 많은 선택적 속성을 가지는 반환 타입과 유니온 타입들은 함수들을 사용하기 어렵게 만든다. **매개변수의 타입의 범위가 넓으면 사용하기 편리**하지만, **반환 타입의 범위가 넓으면 불편**하다.

유니온 타입의 요소별 분기를 위한 방법은 좌표를 위한 기본 형식을 구분하는 것이다. 배열과 배열같은 것을 구분하기 위해서 `LngLat` 와 `LngLatLike` 를 분리하고, `setCamera` 함수가 매개변수를 받을 수 있도록 완전하게 정의된 타입인 Camera 와 Camera 타입이 부분적으로 정의된 버전을 만들 수 있다.

```tsx
interface LngLat { lng: number; lat: number; }
type LngLatLike = LngLat | { long: number; lat: number; } | [number, number];
type LngLatBounds =
  | { northeast: LngLat; southwest: LngLat }
  | [LngLatLike, LngLatLike]
  | [number, number, number, number];

interface Camera {
  center: LngLat;
  zoom: number;
  bearing: number;
  pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
  center?: LngLatLike;
}
/**
interface CameraOptions {
  center?: LngLatLike;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
*/

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;

function focusOnFeature() {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const { center: { lng, lat }, zoom } = camera;
  zoom; // type: number
}
```

`Camera` 타입이 너무 엄격하기 때문에 느슨한 타입인 `CameraOptions` 를 만들었다.  그리고 zoom 타입이 `number | undefined` 가 아닌 `number` 이다.

어쩔 수 없이 다양한 타입을 반환해야 하는 경우가 있다. 하지만 이런 반환 타입이 나쁘다는 것을 잊으면 안된다. 반환 타입과 매개변수를 재활용하기 위해서 **기본 형태와 느슨한 형태를 나눠서 도입**하는 것이 좋다. 그리고 선택적 속성과 유니온 타입은 반환타입보다는 매개변수 타입에서 더 일반적으로 사용되야 한다.
