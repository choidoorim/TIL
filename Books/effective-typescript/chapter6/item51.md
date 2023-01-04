# 아이템51. 의존성 분리를 위해 미러 타입 사용하기

csv 파일을 파싱하는 라이브러리를 사용한다고 가정해보자.

```tsx
function parseCSV(contents: string | Buffer) {
  //...
}
```

Buffer 타입에 대한 정의는 node 타입 선언을 설치해서 얻을 수 있다.

```
$ npm install --save-dev @types/node
```

라이브러리를 공개하게 된다면 타입 선언도 포함하게 된다. 그리고 타입 선언이 `@types/node` 에 의존하기 때문에 `devDependencies` 에 포함된다. devDependencies 에 포함되면 2 가지 문제가 발생한다.

1. @types 와 무관한 자바스크립트 개발자
2. NodeJS 와 무관한 타입스크립트 웹 개발자

1, 2 번의 개발자는 자신들이 사용하지 않는 모듈이 포함되어 있어 혼란스러울 것이다. 따라서 각자가 필요한 모듈만 사용할 수 있도록 **구조적 타이핑을 적용**할 수 있다. @types/node 에 있는 타입을 사용하지 않고, 별도로 필요한 속성만 작성할 수 있다.

```tsx
interface CsvBuffer {
  toString(encoding: string): string;
}
function parseCSV(contents: string | CsvBuffer) {
  //...
}
```

CsvBuffer 타입은 Buffer 타입과 호환되기 때문에 NodeJS 프로젝트에서는 실제 Buffer 인스턴스로 parseCSV 함수를 호출하는 것이 가능하다.

라이브러리가 의존하는 구현과 무관하게 타입에만 의존하는 경우 **필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것(미러링, mirroring)**을 고려해보는 것도 좋다.

이런 방법이 NodeJS 기반 타입스크립트 사용자에게는 변화가 없지만, **그 외 다른 사용자들에게는 더 나은 사양을 제공**할 수 있다. 하지만 다른 라이브러리의 타입 선언을 대부분 추출해야 한다면, `@types/***` 의존성을 추가하는게 더 낫다.
