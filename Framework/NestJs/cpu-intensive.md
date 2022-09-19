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

이 문제를 해결하기 위해서 Javascript 를 병렬로 실행하는 Thread 를 사용해야 합니다.          
[workerpool](https://www.npmjs.com/package/workerpool) 라이브러리는 Thread Pool 패턴을 구현한 Worker Pool 을 쉽게 생성하고 사용할 수 있도록 도와줍니다.

