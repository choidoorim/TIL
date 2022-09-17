# NestJS Custom Exception

기본적으로 예외를 처리하는 방법은 아래 코드의 방법과 같습니다.
표준 예외와 메시지를 통해 충분히 어떤 예외를 발생하는지 추측하고 확인할 수 있다.

```typescript
if (!product) {
  throw new NotFoundException('제품을 찾을 수 없습니다');
}
```

### 1. 가독성
하지만 아래처럼 Custom Exception 을 사용한다면 로직상 가독성을 높일 수 있다.   
Custom Exception 은 네이밍을 통해 일차적으로 어떤 예외가 발생했는지 유추가 가능하다.

```typescript
// exception/product-not-founct-exception.ts
import {NotFoundException} from "@nestjs/common";

export class ProductNotFoundException extends NotFoundException {
    constructor() {
        super('제품을 찾을 수 없습니다');
    }
}

// service logic
if (!product) {
    throw new ProductNotFoundException();
}
```

### 2. 응집도
그리고 인자를 통해 상세한 예외 정보를 제공할 수도 있으며, 
예외에 대한 응집도가 향상되어서 예외에 필요한 메시지, 전달할 정보의 데이터, 데이터 가공 메소드들을 한 곳에서 관리할 수 있게 된다.

```typescript
import { NotFoundException } from "@nestjs/common";

export class ProductNotFoundException extends NotFoundException {
    private static message = '제품을 찾을 수 없습니다';
    
    constructor(id: number) {
        super(`${id} 인 ${ProductNotFoundException.message}`);
    }
}

// service logic
const productId = 1;
if (!product) {
    throw new ProductNotFoundException(productId);
}
```

### 3. 재사용성
Custom Exception 을 만들면 exception 폴더 안에 많은 예외 클래스들이 생기는 단점이 존재하지만
같은 커스텀 예외 클래스가 여러 곳에서 사용될수록 장점은 더욱 극대화 된다. 

### 4. Swagger 의 `@ApiException` 와의 호환성
```typescript
@ApiException(() => [ProductNotFoundException])
@Get('/product')
async getProduct() {
    //...
}
```

참고 자료    
https://nanogiants.github.io/nestjs-swagger-api-exception-decorator/gettingstarted/usage/custom/
https://www.baeldung.com/java-new-custom-exception
https://tecoble.techcourse.co.kr/post/2020-08-17-custom-exception/
