# NestJS - 직렬화(Serializer)
소프트웨어 개발에서 말하는 데이터 직렬화(Serializer) 는 **메모리를 디스크에 저장하거나 네트워크 통신에 사용하기 위한 형식으로 변환하는 것**을 말합니다.   
역직렬화는 그 반대로 **디스크에 저장한 데이터를 읽거나, 네트워크 통신으로 받은 데이터를 메모리에서 쓸 수 있도록 다시 변환하는 것** 입니다.   
즉, 직렬화의 주된 목적은 객체를 상태 그대로 저장하고, 필요할 때 다시 생성하여 사용하는 것 입니다.

![serializer](https://user-images.githubusercontent.com/63203480/148670442-76ca6f52-fc46-4a7f-9cfa-87340fb32cef.png)

## 1. class-transformer 의존성 등록
JSON 직렬화를 위해서는 직렬화 라이브러리가 필요합니다. NestJS 에서는 데코레이터 기반의 직렬화/역직렬화 라이브러리인 ```class-transformer``` 를 사용하고 있습니다.

```npm i class-transformer```

## 2. 글로벌 인터셉터 등록
모든 HTTP 요청에서 직렬화가 가능하도록 ```main.ts``` 파일에 직렬화 인터셉터를 글로벌로 추가합니다.

```typescript
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

위의 선언으로 ```ClassSerializerInterceptor``` 클래스에서 HTTP 응답 값을 중간에서 가로채서(Interceptor) ```class-transformer``` 의 ```classToPlain()``` 함수를 호출하여
JSON 직렬화를 해서 반환합니다.

## 3. DTO 생성

```typescript
import { Exclude } from 'class-transformer';

export class UserResponseDto {
    @Exclude() private readonly _id: number;
    @Exclude() private readonly _name: string;
    @Exclude() private readonly _age: number;
    @Exclude() private readonly _pw: string;

    constructor(user: { name: string; pw: string; id: number; age: number }) {
        this._id = user.id;
        this._name = user.name;
        this._age = user.age;
        this._pw = user.pw;
    }

    @Expose()
    get id(): number {
        return this._id;
    }

    @Expose()
    get name(): string {
        return this._name;
    }

    @Expose()
    get age(): number {
        return this._age;
    }

    get pw(): string {
        return this._pw;
    }
}

```

### Exclude()
노출을 원하지 않는 곳들은 모두 ```Exclude()``` 처리하면 됩니다.

### Expose()
직렬화 대상 필드를 지정합니다.

테스트를 수행해보면 아래와 같이 직렬화 대상 필드는 호출되지만, 노출을 원하지 않는 ```pw``` 필드는 호출되지 않습니다.

```typescript
// controller
//...
@Get('serialization')
serializationTest() {
    return new UserResponseDto({
        id: 1,
        name: 'choi',
        age: 26,
        pw: 'asdasdasd',
    });
}
```

![160450](https://user-images.githubusercontent.com/63203480/148672676-303b0e56-6210-4fc6-8b7c-44a2e4a7b137.png)


출처
https://hub1234.tistory.com/26  
https://jojoldu.tistory.com/610  
https://medium.com/@lunay0ung/basics-%EC%A7%81%EB%A0%AC%ED%99%94-serialization-%EB%9E%80-feat-java-2f3eb11e9a8   
