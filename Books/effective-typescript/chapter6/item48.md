# 아이템48. API 주석에 TSDoc 사용하기

```tsx
// 인사말을 생성하고, 결과는 보기 좋게 꾸며진다
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

위 함수의 앞부분에 주석이 있기 때문에 함수가 어떤 기능을 하는지 쉽게 알 수 있다. 그러나 사용자를 위한 것이라면 JSDoc 스타일의 주석으로 만드는 것이 좋다.

```tsx
/** 인사말을 생성하고, 결과는 보기 좋게 꾸며진다 */
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

대부분의 편집기에서는 함수가 호출되는 곳에서 함수에 붙어 있는 JSDoc 스타일의 주석을 툴팁으로 표시해주지만 인라인 주석을 표시해주지 않는다.

타입스크립트 언어 서비스에서는 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋다. 공개 API 에 주석을 붙인다면 JSDoc 형태로 작성해야 한다.

aws sdk config 파일도 실제로 아래와 같이 TSDoc 가 적용된 것을 확인할 수 있다.

```tsx
export interface APIVersions {
    /**
     * A string in YYYY-MM-DD format that represents the latest possible API version that can be used in all services (unless overridden by apiVersions). Specify \'latest\' to use the latest possible version.
     */
    apiVersion?: "latest"|string;
    /**
     * A map of service identifiers (the lowercase service class name) with the API version to use when instantiating a service. Specify 'latest' for each individual that can use the latest available version.
     */
    apiVersions?: ConfigurationServiceApiVersions;
```

`@param, @returns` 같은 일반적 규칙을 사용할 수 있다. 타입스크립트 관점에서는 TSDoc 라고 부르기도 한다.

```tsx
/**
 *  인사말을 생성하고, 결과는 보기 좋게 꾸며진다
 *  @param name 인사할 사람의 이름
 *  @param title 그 사람의 칭호
 *  @return 사람이 보기 좋은 형태의 인사말
*/
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

`@param, @returns` 를 추가하면 함수를 호출하는 부분에서 각 매개변수와 관련된 설명을 보여준다.
