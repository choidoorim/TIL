# NestJS - numeric query parameter
개발을 하던 도중, query parameter 에 대한 validation 처리를 하려고 dto 파일을 만들고 있었는데

아래와 같은 오류가 발생했습니다.

```
// xxx.dto.ts
import { IsNumber } from 'class-validator';

export class PaginationDto {
  @IsNumber()
  private readonly pageNo: number;

  @IsNumber()
  private readonly pageSize: number;

  getSkip(): number {
    return (this.pageNo - 1) * this.pageSize;
  }

  getTake(): number {
    return this.pageSize;
  }
}
```

![캡처](https://user-images.githubusercontent.com/63203480/144051446-7758b533-a8a6-48b2-9d7d-4bf65ed1c222.PNG)

분명히 query parameter 에 number 를 썼는데 왜 에러가 발생하는지 궁금했습니다.

그리고 console.log 를 통해 query parameter 의 결과를 출력해봤는데... 문자열 타입으로 출력되는 것을 확인할 수 있었습니다. 원래 query parameter 의 결과가 문자열 타입인지는 찾아봐야할 것 같습니다.

![캡처2](https://user-images.githubusercontent.com/63203480/144051482-33b18d06-6c65-48b4-a829-27d2e078557b.PNG)

그래서 열심히 찾아봤는데 해당 [링크](https://dev.to/avantar/validating-numeric-query-parameters-in-nestjs-gk9) 에 해결방법을 설명해준 글이 있었습니다.

바로 **class-transformer 라이브러리**를 사용하는 방법입니다. 만약 class-transformer 라이브러리가 설치되어있지 않다면 설치해줘야 합니다.

[https://www.npmjs.com/package/class-transformer](https://www.npmjs.com/package/class-transformer)

간략히 설명하자면 해당 라이브러리가 제공하는 기능을 통해 형 변환을 가능하게 해줍니다.

따라서 아래와 같이 코드를 변환하여 문제를 해결했습니다.

```
import { IsNumber } from 'class-validator';
import { Type } from 'class-transformer';	// *************

export class PaginationDto {
  @IsNumber()
  @Type(() => Number)	// *************
  private readonly pageNo: number;

  @IsNumber()
  @Type(() => Number)	// *************
  private readonly pageSize: number;

  getSkip(): number {
    return (this.pageNo - 1) * this.pageSize;
  }

  getTake(): number {
    return this.pageSize;
  }
}
```