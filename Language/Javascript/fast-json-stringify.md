# Json Stringify: fast-json-stringify 라이브러리 사용하기
[fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) 는 object 를 string 으로 변환해주는 라이브러리로 입니다.
기존 Javascript 에서 제공하는 ```JSON.stringify()``` 보다 작은 양의 Payload 일 경우 훨씬 더 빠르다고 합니다.    

Payload 가 커지면 커질 수록 성능의 이점이 줄어든다고 하니 상황에 따라 잘 사용하는 것이 좋습니다.

## example
```email, age, phoneNumber``` 3 개의 Json 형식의 데이터가 string 으로 변환해봤습니다.

```typescript
import * as fastJson from 'fast-json-stringify';

export const stringifyUserInfo = fastJson({
  title: 'stringify Test',
  type: 'object',
  properties: {
    email: {
      type: 'string',
    },
    age: {
      type: 'integer',
    },
    phoneNumber: {
      type: 'string',
    },
  },
});
```

```typescript
@Post()
async stringifyTest(@Body() user: userInterface) {
    const userStringInfo = stringifyUserInfo(user);
    console.log(typeof userStringInfo);
    return userStringInfo;
}
```

#### result

```json
{
    "email": "asdasd",
    "age": 10,
    "phoneNumber": "0101010101010"
}
```

![image](https://user-images.githubusercontent.com/63203480/149657118-bc594e7e-4a43-4682-ae69-3b9f8e0ec4fa.png)

![image](https://user-images.githubusercontent.com/63203480/149657154-a01c6aa4-8ad1-40d8-8225-d294a188da53.png)

JSON 형태의 데이터가 정상적으로 String 으로 변환 된 것을 확인할 수 있습니다.

## required
```string``` 으로 변환할 객체의 필수 값을 설정할 수 있는 옵션입니다.
만약 필수 값으로 설정한 데이터가 없을 경우 **에러**를 return 합니다.

```json
{
    "age": 10,
    "phoneNumber": "0101010101010"
}
```

![image](https://user-images.githubusercontent.com/63203480/149657468-3b53d80c-71ba-4c2e-b99c-e478be64d89a.png)


[공식문서](https://www.npmjs.com/package/fast-json-stringify) - 해당 라이브러리는 다양한 기능들을 제공하기에 필요에 따라 잘 사용하면 좋을 것 같습니다.