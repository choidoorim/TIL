# Typescript 로 알아보는 SOLID(객체지향 설계)
주로 Express 를 통해 서버를 개발하다가, 최근 NestJS 를 사용하고 공부하다 보니 객체지향 설계에 대해 더욱 궁금하고 깊게 알고 싶어져 정리하게 되었습니다.  

SOLID 디자인 원칙은 더 나은, 더 깔끔한 코드를 작성하는 방법으로 **Robert C. Martin** 이라는 소프트웨어 엔지니어가 발명했습니다.  

SOLID 원칙을 Typescript 를 통해 알아볼려고 합니다.

## Single Responsibility Principle(SRP) - 단일 책임 원칙
### 어떤 클래스를 변경해야하는 이유는 단 하나뿐이여야 한다.
클래스는 하나의 목적과 책임을 가지고 있어야하기에, 변경하기 위한 이유도 하나이어야 한다. 
이 원칙을 따르게 된다면 코드를 더 잘 유지 관리하고, 잠재적인 부작용을 최소화할 수 있습니다.

나쁜 예로 책이라는 클래스가 있고, 이 클래스에는 책의 title, author 등의 모델링과 파일을 저장하는 책임이 존재하는 것을 확인할 수 있습니다.

```typescript
class Book {
    public title: string;
    public author: string;
    public description: string;
    public pages: number;

    public saveToFile(): void {
        // ...
    }
}
```

아래의 예제는 단일 책임 원칙에 맞춰 위의 나쁜 예를 바꾼 것입니다. 목적 별로 하나씩 클래스를 같게 되는 것을 확인할 수 있습니다.

```typescript
class Book {
    public title: string;
    public author: string;
    public description: string;
    public pages: number;
}

class Persistence {
    public saveToFile(book: Book): void {
        // ...
    }
}
```


## Open-Closed Principle(OCP) - 개방-폐쇠 원칙
## Liskov Substitution Principle(LSP) - 리스코프 치환 원칙
## Interface Segregation Principle(ISP) - 인터페이스 분리 원칙
## Dependency Inversion Principle(DIP) - 의존관계 역전 원칙

https://blog.bitsrc.io/solid-principles-in-typescript-153e6923ffdb