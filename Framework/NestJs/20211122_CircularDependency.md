# Nest.JS Circular Dependency(순환 종속성) 문제

NestJS 를 통해 프로젝트 구현중에 아래 사진과 같은 문제가 발생했다.
![캡처](https://user-images.githubusercontent.com/63203480/142796344-ef06d178-fd5d-4e74-9150-2153b038c780.PNG)

### Circular Dependency(순환 종속성)
[공식문서](https://docs.nestjs.com/fundamentals/circular-dependency) 를 확인해보면 순환 종속성은 클래스 A 에 클래스 B 가 필요하고, 클래스 B 에 클래스 A 가 필요할 때 발생한다고 합니다.
가능한 최대한 순환종속성은 피해야 하지만 항상 그렇게 개발할 수는 없기에 NestJS 에서는 해결 방안을 제공해줍니다.

### 해결방법: 전달 참조(Forward Reference)
```@nestjs/common``` 에서 패키지로 제공해주는 ```forwardRef()``` 기능을 사용하면 해결할 수 있습니다.
클래스에 중첩을 허용할 수 있도록 하는 유틸리티 기능입니다.

```typescript
// moduleA
@Module({
    imports: [ModuleB],
    ///...
})

// moduleB
@Module({
    imports: [ModuleA],
    ///...
})
```

만약 위의 코드처럼 moduleA 와 moduleB 가 서로 종속적이라고 가정한다면, 
아래와 같이 ```forwardRef()``` 를 사용하여 코드를 변경하면 문제를 해결할 수 있다.

```typescript
// moduleA
@Module({
    imports: [forwardRef(()=> ModuleB)],
    ///...
})

// moduleB
@Module({
    imports: [forwardRef(()=> ModuleA)],
    ///...
})
```

일단은 ```forwardRef()``` 의 도움을 받아서 해결은 했는데, 사용하지 않을 방법이 있는지 의존성에 대해서 공부를 해봐야할 것 같다.