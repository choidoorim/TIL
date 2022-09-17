# NestJS Custom Exception

기본적으로 예외를 처리하는 방법은 아래 코드의 방법과 같습니다.
표준 예외와 메시지를 통해 충분히 어떤 예외를 발생하는지 추측하고 확인할 수 있습니다.

```typescript
if (!product) {
  throw new NotFoundException('제품을 찾을 수 없습니다');
}
```

### 1. 가독성
하지만 아래처럼 Custom Exception 을 사용한다면 로직상 가독성을 높일 수 있습니다..  
Custom Exception 은 클래스의 네이밍을 통해 일차적으로 어떤 예외가 발생했는지 유추가 가능합니다.

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

### 3. 재사용성 및 유지보수
Custom Exception 을 만들면 exception 폴더 안에 많은 커스텀 예외 클래스들이 생기는 단점이 존재하지만
같은 커스텀 예외 클래스가 여러 곳에서 사용될수록 장점은 더욱 극대화 됩니다.

물론 Message 에 대한 내용을 담은 클래스, 모듈을 만들어서 재사용한다면 Custom Exception 을 만드는 것만큼 재사용성 측면에서 좋겠지만,
```typescript
// product-error.ts
export class ProductError {
    static PRODUCT_NOT_FOUND = '제품을 찾을 수 없습니다';
}

// service logic
if (!product) {
    throw new NotFoundException(ProductError.PRODUCT_NOT_FOUND);
}
```
위의 가독성, 응집도 등의 특징들을 생각한다면 Custom Exception 을 사용하는 방법이 더 좋은 방법이라는 생각이 듭니다.

그리고 만약 어떤 로직에 대한 공통되는 Exception 이 `NotfoundException` 에서 `BadRequestException` 으로 바껴야 되는 상황이 온다면
표준 예외를 사용한다면 변경되는 예외에 대한 모든 로직을 찾아서 수정해줘야 하는 단점도 있을 것이라고 생각됩니다.

#### 표준 예외 - 직접 변경
```typescript
// service logic1
if (!product) {
    // throw new NotFoundException(ProductError.PRODUCT_NOT_FOUND);
    throw new BadRequestException(ProductError.PRODUCT_NOT_FOUND);
}

// service logic2
if (!product) {
    // throw new NotFoundException(ProductError.PRODUCT_NOT_FOUND);
    throw new BadRequestException(ProductError.PRODUCT_NOT_FOUND);
}
```

#### 커스텀 예외 - 커스텀 예외 클래스만 수정
```typescript
import { BadRequestException } from "@nestjs/common";

// export class ProductNotFoundException extends NotFoundException {
export class ProductNotFoundException extends BadRequestException {
    private static message = '제품을 찾을 수 없습니다';
    
    constructor(id: number) {
        super(`${id} 인 ${ProductNotFoundException.message}`);
    }
}

// service logic1
if (!product) {
    throw new ProductNotFoundException(ProductError.PRODUCT_NOT_FOUND);
}

// service logic2
if (!product) {
    throw new ProductNotFoundException(ProductError.PRODUCT_NOT_FOUND);
}
```


### 4. Swagger 의 `@ApiException` 와의 호환성
```typescript
@ApiException(() => [ProductNotFoundException])
@Get('/product')
async getProduct() {
    //...
}
```

표준 예외를 통해 충분히 어떤 상황의 예외인지 유추가 가능하고 알 수 있기 때문에 
커스텀 예외를 만들어서 사용하는 방법이 무조건 정답이라고 할 수는 없을 것 같습니다.

따라서 팀원, 동료들과 충분한 얘기를 나눈 뒤 커스텀 예외를 사용하는 것이 좋아보입니다.

참고 자료    
https://nanogiants.github.io/nestjs-swagger-api-exception-decorator/gettingstarted/usage/custom/
https://www.baeldung.com/java-new-custom-exception
https://tecoble.techcourse.co.kr/post/2020-08-17-custom-exception/
