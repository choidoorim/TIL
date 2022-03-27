# Typescript - 배열 내에서 undefined 제거하기 (User-Defined Type Guards)

```typescript
const arr: (string | undefined)[] = ['test1', 'test2', undefined, 'test3', 'test4'];
```

위와 같은 배열에서 undefined 를 제거하기 위해서는 ```filter()``` 메서드를 사용하는 방법이 있습니다.

```typescript
const arr: (string | undefined)[] = ['test1', 'test2', undefined, 'test3', 'test4'];

const filter_undefined_arr = arr.filter((data) => data !== undefined); // ['test1', 'test2', 'test3', 'test4']
```

결과는 undefined type 이 제거된 배열이 return 되는 것을 확인할 수 있습니다. 하지만 여기서 문제가 하나 발생합니다.
filter 는 배열 안에 요소를 걸러주지만 type 을 함께 걸러주지는 않습니다.
따라서 위의 예제의 ```filter_undefined_arr``` 변수의 type 은 여전히 ```(string | undefined)[]``` 으로 바뀌지 않은 것입니다.

https://stackoverflow.com/questions/43010737/way-to-tell-typescript-compiler-array-prototype-filter-removes-certain-types-fro

https://github.com/microsoft/TypeScript/issues/20812

## 해결방법: User-Defined Type Guards(사용자 정의 Type Guards)
Typescript 에서는 사용자 정의 Type Guards 함수를 만들어서 위의 문제를 해결할 수 있습니다.
**"어떤 인자명은 어떠한 타입이다"** 라는 값을 return 하는 함수입니다.

```typescript
const isDefined = <T>(arg: T | undefined): arg is T => arg !== undefined; // 인자로 넘어온 매개변수가 undefined 가 아닐 때 true 를 return 하는 함수입니다. 
```

- ```<T>``` : 제네릭 타입으로 컴포넌트가 여러 타입에서 작동할 수 있게 도와준다.

이러한 사용자 정의 Type Guards 함수를 만들어서 filter 함수에 적용시켜주면 됩니다.  

함수를 만들지 않는 방법도 존재합니다.

```typescript
const filter_undefined_arr = arr.filter((arg): arg is string => arg !== undefined);
```

```arg``` 의 타입을 string 으로 인식하게 하는 것 입니다.

### Full Code
```typescript
const arr: (string | undefined)[] = ['test1', 'test2', undefined, 'test3', 'test4'];

const isDefined = <T>(arg: T | undefined): arg is T => arg !== undefined;
const filter_undefined_arr = arr.filter(isDefined);
// const filter_undefined_arr = arr.filter((arg): arg is string => arg !== undefined);
```

개인적으로 filter 함수를 사용해서 undefined 라던지 null 등의 타입을 걸러내는 기능들이 여러 코드에서 사용된다면 공통된 하나의 메서드를 만들어서 재사용하는 방법이 더 좋고 깔끔해지는 것 같습니다.
