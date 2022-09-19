# NodeJS, NestJS - Cpu Intensive 한 작업 처리하기
NodeJS 는 싱글 스레드지만 비동기 + Non-Blocking 의 특징을 가지고 있기 때문에
I/O 작업이 빈번한 경우에는 좋지만 **싱글스레드이기 때문에 CPU Intensive 한 작업을 처리하기에는 좋지 않다**는 특징을 가지고 있습니다.

만약 Node 기반의 서버가 CPU Intensive 한 작업을 할 경우 어떻게 될까요?
```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('cpu-intensive')
export class CpuIntensiveController {
  private calCount(num = 2_000_000_000_000) {
    let count = 1;
    for (let i = 0; i < num; i++) {
      count += 1;
    }
    return count;
  }

  @Get('/')
  getCount() {
    return this.calCount();
  }

  @Get('/hello')
  getHello() {
    return 'hello';
  }
}
```
만약 위의 로직에서 2개의 API(`/cpu-intensive -> /cpu-intensive/hello`) 를 순서대로 호출하게 되면 
`/cpu-intensive` 의 작업이 끝날 때까지 `/cpu-intensive/hello` 에 대한 응답은 받을 수 없습니다.

<img width="766" alt="스크린샷 2022-09-19 오후 11 54 25" src="https://user-images.githubusercontent.com/63203480/191047608-95260cd9-3562-4f29-8bdf-56e87cbb92a7.png">

이 문제를 해결하기 위해서 Javascript 를 다중 스레드로 사용할 수 있게 해야합니다.          
[workerpool](https://www.npmjs.com/package/workerpool) 라이브러리는 Thread Pool 패턴으로 구현한 Worker Pool 을 쉽게 생성하고 사용할 수 있도록 도와줍니다.

```typescript
import { Controller, Get } from '@nestjs/common';
import * as workerpool from 'workerpool';

@Controller('cpu-intensive')
export class CpuIntensiveController {
  private pool = workerpool.pool();

  private calCount = (num: number) => {
    let count = 1;
    for (let i = 0; i < num; i++) {
      count += 1;
    }
    return count;
  };

  @Get('/')
  getCount() {
    const count = this.pool.exec(this.calCount, [20_000_000_000]);
    this.pool.terminate();
    return count;
  }

  @Get('/hello')
  getHello() {
    return 'hello';
  }
}
```

worker pool 라이브러리를 적용시킨 뒤 다시 2개의 API(`/cpu-intensive -> /cpu-intensive/hello`) 를 순서대로 호출하게 되면
`/cpu-intensive/hello` API 는 `/cpu-intensive` API 가 끝나는 것을 기다리지 않고 응답을 받을 수 있는 것을 확인할 수 있습니다.

<img width="672" alt="스크린샷 2022-09-20 오전 12 34 53" src="https://user-images.githubusercontent.com/63203480/191056367-afd7f5b7-77b7-4ecb-8b97-28d4e6a88454.png">

<img width="508" alt="스크린샷 2022-09-20 오전 12 35 00" src="https://user-images.githubusercontent.com/63203480/191056401-c622c46b-0074-4239-b54e-653c4db4d120.png">


이렇게 worker pool 을 통해 NodeJS 환경에서도 Cpu Intensive 한 작업도 해결할 수 있습니다.
