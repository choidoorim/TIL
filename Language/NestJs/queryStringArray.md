# NestJS : Query String 배열로 처리하기

#### endPoint
```text
/test?queryData=asdasd&queryData=bbbbefe ...
```

#### controller
```javascript
@Get('/test')
queryStringArrayTest(@Query('queryData') queryData: string[]) {
    return queryData;
}
```

우선 위 처럼 할 경우 query String 을 배열로 처리할 수 있습니다.

#### Result
```json
[
    "asdasd",
    "bbbbefe"
]
```

endPoint 의 Query String 를 위와 같이 처리한다면 ```queryData``` 가 반복돼서 사용되고, 길어질 수록 보기 좋지 않을 수 있습니다.

하지만 ```class-validator, class-transformer``` 라이브러리를 통해 검증 가능하고, 깔끔하게 query String 을 사용할 수 있는 방법이 있습니다.


## "{endpoint}?query=A,B,C..." 형식으로 처리하기
### Dto
class-validator 에서 데코레이더 옵션을 ```{each: true}``` 로 해서 DTO 를 만들어 줍니다.
```{each: true}``` 옵션은 배열을 검증하기 위해 사용하는 옵션입니다.
class-transformer 의 **Transformer** 를 사용해서 QueryString 의 데이터를 변환시킬 수 있습니다. 

그리고 개별적인 검증이 가능한지 테스트 하기 위해서 class-validator 의 ```Length``` 를 사용해보겠습니다.

#### query-string-array-test.dto.ts
```typescript
import { Transform } from 'class-transformer';
import { IsString, Length } from 'class-validator';

export class QueryStringArrayTestDto {
    @Transform(({ value }) => value.split(','))
    @IsString({ each: true })
    @Length(1, 5, { each:true })
    readonly queryData: string[];
}
```

#### controller
```typescript
@Get()
queryStringArrayTest(@Query() queryData: QueryStringArrayTestDto) {
    return queryData;
}
```

이렇게 dto 와 controller 를 작성해서 테스트 해보겠습니다. 만약 각각의 데이터 중 String 의 길이가 5 이상일 경우 어떻게 될까요?

#### endpoint
```text
{endpoint}?queryData=test,testString
```

#### Result
```json
{
    "statusCode": 400,
    "message": [
        "each value in queryData must be longer than or equal to 1 and shorter than or equal to 5 characters"
    ],
    "error": "Bad Request"
}
```

```testString``` 이 5개 이상의 문자열이기 때문에 400번 Error 가 발생하게 됩니다.    
1 에서 5 사이의 문자열일 경우는 아래와 같이 정상적으로 작동되는 것을 확인할 수 있습니다.

#### Result
```json
{
    "queryData": [
        "test1",
        "test2"
    ]
}
```

이렇게 NestJS 에서 ```class-validator, class-transformer``` 라이브러리를 사용해서 Query String 을 배열로 보기 좋게 처리해봤습니다. 