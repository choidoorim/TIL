# NestJS - Interface Dependency Injection(DI)
NestJS 는 Spring 처럼 기본적으로 생성자 기반의 DI(Dependency Injection) 을 지원합니다.  
DI 는 Ioc 컨테이너에 객체의 인스턴스화를 위임하는 역전의 제어(Inversion of Control) 기술입니다.

그리고 의존성 주입(DI) 은 객체지향의 5 대 원칙 중 DIP(의존 관계 역전 원칙) 를 따르는 방법 중에 하나입니다.
또한 중요한 또 다른 원칙이 있는데 바로 OCP(개방폐쇄원칙) 입니다.

- DIP: "추상화에 의존해야지 구체화에 의존하면 안된다"
- OCP: "확장에는 열려있으나 변경에는 닫혀있어야 한다"

그렇다면 어떻게 설계해야 이 원칙을 잘 지킬 수 있을까요?     
아래의 예제를 통해 알아봅시다.

아래 예제는 기본적으로 NestJS 에서 DI 를 사용하는 방법입니다.
#### Module
```typescript
import { Module } from '@nestjs/common';
import { CarsController } from './cars.controller';
import { AvanteService } from './avante.service';

@Module({
  controllers: [CarsController],
  providers: [AvanteService],
})
export class CarsModule {}
```

#### Controller
```typescript
import { Controller, Get } from '@nestjs/common';
import { AvanteService } from './avante.service';

@Controller('cars')
export class CarsController {
  constructor(private readonly carsService: AvanteService) {}

  @Get()
  pressAcceleratorAndGetSpeed() {
    this.carsService.accelerator(3);
    this.carsService.accelerator(6);

    return this.carsService.getSpeed();
  }
}
```

#### Service
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AvanteService {
  private speed = 1;

  accelerator(press: number): void {
    this.speed = this.speed * press;
  }

  getSpeed(): number {
    return this.speed;
  }
}
```
```AvanteService```에 선언된  ```Injectable``` 데코레이터가 NestJS Ioc 컨테이너에서 관리할 수 있는 클래스롤 선언합니다.
그리고 ```CarsController``` 에서는 생성자 주입을 통해 ```AvanteService``` 에 대한 Dependency 를 선언했습니다.

위 코드는 이미 **"프로그래머는 추상화에 의존해야지 구체화에 의존하면 안된다"** 라는 DIP 에 위반됩니다.
```typescript
constructor(private readonly carsService: AvanteService) {}
```
왜냐하면 현재 Controller 클래스에서 ```AvanteService``` 라는 **구현 클래스에 의존**하고 있기 때문입니다.

또한 Controller 에서 ```AvanteService``` 가 아닌 새로운 ```TeslaService``` 사용하기 위해서는 아래와 같이 Controller 클래스에 생성자를 변경시켜야 합니다.
```typescript
// Controller
// constructor(private readonly carsService: AvanteService) {}
constructor(private readonly carsService: TeslaService) {}
//...
``` 
이렇게 된다면 ```CarsController``` "클래스는 **"확장에는 열려있지만 변경에는 닫혀있어야 한다"** 라는 OCP 원칙에 위반됩니다. 
```TeslaService``` 로 바꿔주는 과정에서 **Controller 클래스를 수정**했기 때문입니다.

## 1. Interface Dependency Injection

객체지향의 원칙을 지키지못하는 문제들은 Interface Dependency Injection 을 해결할 수 있습니다.      
Interface 를 아래와 같이 만들어서 Service 클래스에서 implements 를 통해 인터페이스를 구현하도록 합니다.

한 가지 아쉬운 점이 있다면 Typescript 에서는 런타임 코드에 Interface 가 존재하지 않습니다.     
즉, Typescript 의 ```Types/Interface``` 는 컴파일이 되면 사라지기 때문에 Interface 를 NestJS 에서 사용하지 못하는 것입니다.          
따라서 NestJS ```Providers``` 의 ```provider``` 에서 토큰을 넣어주어 구현체 클래스와 연결하여 해결해줘야 합니다.

#### Interface
[Symbol](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 을 통해 "CarsInterface" 라는 고유한 값을 만들어줍니다.
```typescript
export const ICarsSymbol = Symbol('CarsInterface');

export interface CarsInterface {
  accelerator(press: number): void;

  getSpeed(): number;
}
```

#### Service
```typescript
import { Injectable } from '@nestjs/common';
import { CarsInterface } from './cars-interface';

@Injectable()
export class AvanteService implements CarsInterface {
  private speed = 1;

  accelerator(press: number): void {
    this.speed = this.speed * press;
  }

  getSpeed(): number {
    return this.speed;
  }
}
```

#### Module
```typescript
import { Module } from '@nestjs/common';
import { CarsController } from './cars.controller';
import { AvanteService } from './avante.service';
import { ICarsSymbol } from './cars-interface';

@Module({
  controllers: [CarsController],
  providers: [
    {
      provide: ICarsSymbol,
      useClass: AvanteService,
    },
  ],
})
export class CarsModule {}
```

#### Controller
```typescript
import { Controller, Get, Inject } from '@nestjs/common';
import { CarsInterface, ICarsSymbol } from './cars-interface';

@Controller('cars')
export class CarsController {
  constructor(
    @Inject(ICarsSymbol) private readonly carsService: CarsInterface,
  ) {}

  @Get()
  pressAcceleratorAndGetSpeed() {
    this.carsService.accelerator(3);
    this.carsService.accelerator(6);

    return this.carsService.getSpeed();
  }
}
```

구현체에 의존하는 것이 아닌 인터페이스에 의존하도록 설계하여 **역할과 구현을 분리**하고, **유연하게 구현체를 변경할 수 있게 시스템을 설계**할 수 있습니다.   
만약 ```Avante``` 클래스가 아닌 ```Tesla``` 클래스로 바꿀려면 어떻게 해야할까요?
implements 를 통해 Interface 를 구현한 Tesla 클래스를 모듈에서 변경해주기만 하면 됩니다.

#### service
```typescript
import { Injectable } from '@nestjs/common';
import { CarsInterface } from './cars-interface';

@Injectable()
export class TeslaService implements CarsInterface {
  private speed = 3;

  accelerator(press: number): void {
    this.speed = this.speed * press;
  }

  getSpeed(): number {
    return this.speed;
  }
}
```

#### module
```typescript
import { Module } from '@nestjs/common';
import { CarsController } from './cars.controller';

// interface
import { CarsInterface } from './cars-interface';

// use-case
import { AvanteService } from './avante.service';
import { TeslaService } from './tesla.service';

@Module({
  controllers: [CarsController],
  providers: [
    {
      provide: ICarsSymbol,
      // useClass: AvanteService,
      useClass: TeslaService,
    },
  ],
})
export class CarsModule {}
```

결과적으로 Controller 클래스의 생성자의 변경이 필요 없게되므로 OCP 원칙을 지킬 수 있게 되었습니다.
또한 인터페이스에 의존하고 있기 때문에 DIP 원칙도 지킬 수 있습니다.

하지만 이 방법은 생성자에 ```Inject``` 데코레이터를 사용해서 주입해야 해야합니다. 
그리고 Javascript 토큰을 이용해서 사용하기 때문에 문제가 해결은 되었지만 실제로 인터페이스와는 관련이 없습니다.
또한 토큰을 수동으로 주입해야 하기 때문에 귀찮고 번거로운 작업일 수도 있습니다.     

## 2. 추상 클래스(Abstract Class) Dependency Injection
이러한 토큰을 수동으로 주입하는 것을 해결할 수 있는 다른 방법이 있습니다. 바로 런타임에 참조되는 추상 클래스를 사용하는 것입니다.      
인터페이스를 추상 클래스로 변경하고, 모듈 클래스에서 추상 클래스를 ```Providers``` 의 ```provider``` 에 설정해주면 됩니다.      
그렇게 된다면 최종적으로 아래의 코드와 같이 변경이 됩니다.

#### Abstract Class
```typescript
export abstract class CarsInterface {
  abstract accelerator(press: number): void;

  abstract getSpeed(): number;
}
```

#### Controller
```typescript
import { Controller, Get } from '@nestjs/common';
import { CarsInterface } from './cars-interface';

@Controller('cars')
export class CarsController {
  constructor(private readonly carsService: CarsInterface) {}

  @Get()
  pressAcceleratorAndGetSpeed() {
    this.carsService.accelerator(3);
    this.carsService.accelerator(6);

    return this.carsService.getSpeed();
  }
}
```

#### Module
````typescript
import { Module } from '@nestjs/common';
import { CarsController } from './cars.controller';

// interface
import { CarsInterface } from './cars-interface';

// use-case
import { AvanteService } from './avante.service';
import { TeslaService } from './tesla.service';

@Module({
  controllers: [CarsController],
  providers: [
    {
      provide: CarsInterface,
      useClass: TeslaService,
    },
  ],
})
export class CarsModule {}
````

최종적으로 토큰을 수동으로 ```Inject``` 데코레이터에 주입하는 번거로운 작업을 해결할 수 있습니다.   
두 가지 방법 모두 OCP, DIP 원칙을 지킬 수 있는 좋은 해결방법이라고 생각이 되기 때문에 선택해서 사용하면 좋을 것 같습니다.
