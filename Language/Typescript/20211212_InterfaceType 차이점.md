# Typescript Interface 와 Type 차이점
타입스크립트를 통해 개발하다보면 **Type Alias** 와 **Interface** 중 어는 것을 사용해서 Type 을 정의해야할까 라는 문제가 생깁니다.

## 공통점
### 객체 타입에 대한 이름 지어주기

#### Interface
```typescript
interface UserInterface {
    name: string
    age: number 
}

const user: UserInterface = {
    name: 'choi',
    age: 100
}
```

#### Type Alias
```typescript
type UserTypeAlias = {
    name: string,
    age: number
}

const user: UserTypeAlias = {
    name : 'choi',
    age: 100
}
```

## 차이점
### 1. 확장 하는 법
```Type Alias``` 와 ```Interface``` 는 모두 **Interface 에 대한 extends** 와 **class 에 대한 implements 키워드를 사용하여 확장이 가능**합니다.

#### Interface, Class 에 대한 확장
```typescript
interface UserInterface {
    name: string
    age: number
}

type UserTypeAlias = {
    name: string
    age: number
}

// extends 
interface UserInterface2 extends UserInterface {
    region: string
}

interface UserTypeAlias2 extends UserTypeAlias {
    region: string
}

// implements
class ClassInterface implements UserInterface2 {
    constructor(public name: string, public age: number, public region: string) {}
}

class ClassTypeAlias implements UserTypeAlias2 {
    constructor(public name: string, public age: number, public region: string) {}
}
```

#### TypeAlias 에 대한 확장
TypeAlias 로 만들어진 정적 타입에 extends, implements 키워드는 사용하지 못합니다.    
대신 ```&``` 을 통해 확장을 해야 합니다.

```typescript
type UserTypeAlias = {
    name: string
    age: number
}

type UserTypeAlias2 = UserTypeAlias & {
    region: string
}

const User: UserTypeAlias2 = {
    name: 'choi',
    age: 100,
    region: '경기도'
};
```

### 2. 선언 병합
Type Alias 와 Interface 의 중요한 차이점은 **Type Alias 는 새로운 속성을 추가하기 위해서 다시 같은 이름으로 선언할 수 없지만, Interface 는 항상 선언적 확장이 가능**하다는 것입니다.

### Interface
같은 이름의 Interface 를 선언 시 자동으로 확장이 됩니다.
```typescript
interface User {
    name: string
}

interface User {
    age: number
}

const user: User = {
    name: 'choi',
    age: 100,
}
```

### Type Alias
```typescript
type User = {
    name: string
}

type User = {
    age: number
}
// ERROR - TS2300: Duplicate identifier 'User'.
```

## 3. computed value 사용
**Type Alias 는 가능**하지만, Interface 는 불가능합니다.
### Type Alias
```typescript
type names = 'firstName' | 'lastName';

type NameTypes = {
    [key in names]: string
}

const userName: NameTypes = {
    firstName: 'choi',
    lastName: 'durumi',
}
```

### Interface
```typescript
type names = 'firstName' | 'lastName';

interface NameTypes {
    [key in names]: string
}
// ERROR - TS1169: ~~
```

## 마무리
Type Alias 와 Interface 에 대해서 알아봤다. 
결론적으로 둘 중 어느것을 사용할지는 통일이 필요한 것 같고, 타입 간 합성이나 객체에서만 쓴다면 Interface 가 더 좋아보인다.


참고
https://medium.com/humanscape-tech/type-vs-interface-%EC%96%B8%EC%A0%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-f36499b0de50    
https://yceffort.kr/2021/03/typescript-interface-vs-type