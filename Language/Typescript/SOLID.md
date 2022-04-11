# Typescript 로 알아보는 SOLID(객체지향 설계)
주로 Express 를 통해 서버를 개발하다가, 최근 NestJS 를 사용하고 공부하다 보니 객체지향 설계에 대해 더욱 궁금하고 깊게 알고 싶어져 정리하게 되었습니다.  

SOLID 디자인 원칙은 더 나은, 더 깔끔한 코드를 작성하는 방법으로 **Robert C. Martin** 이라는 소프트웨어 엔지니어가 발명했습니다.  

SOLID 원칙을 Typescript 를 통해 알아볼려고 합니다.

## 1. Single Responsibility Principle(SRP) - 단일 책임 원칙
### "어떤 클래스를 변경해야하는 이유는 단 하나뿐이여야 한다"
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


## 2. Open-Closed Principle(OCP) - 개방-폐쇠 원칙
### "소프트웨어 엔티티는 확장을 위해서는 열려 있어야 하지만, 수정을 위해는 닫혀 있어야 한다"
클래스를 오버라이딩 하기보다는 확장하는 것이 더 좋습니다. 인터페이스나 클래스를 구현해서 이전 코드를 건드리지 않고 새로운 기능으로 쉽게 확장할 수 있어야 합니다.

아래의 3 개의 Class 중 AreaCalculator 라는 클래스를 사용하여 Rectangle 및 Circle 클래스의 면적을 계산합니다. 아래의 예제는 잘못된 방법입니다.

```typescript
class Rectangle {
    public width: number;
    public height: number;

    constructor(width: number, height: number) {
        this.width = width;
        this.height = height;
    }
}

class Circle {
    public radius: number;

    constructor(radius: number) {
        this.radius = radius;
    }
}

class AreaCalculator {
    public calculateRectangleArea(rectangle: Rectangle): number {
        return rectangle.width * rectangle.height;
    }

    public calculateCircleArea(circle: Circle): number {
        return Math.PI * (circle.radius * circle.radius);
    }
}
```

개방폐쇠 원칙을 따르기 위해서 Shape 이라는 interface 를 추가해주면 됩니다. 
따라서 모든 Shape 관련 클래스들은 Shape 인터페이스를 구현해서 이 인터페이스에 의존할 수 있습니다.

```typescript
interface Shape {
    calculateArea(): number;
}

class Rectangle implements Shape {
    public width: number;
    public height: number;

    constructor(width: number, height: number) {
        this.width = width;
        this.height = height;
    }

    public calculateArea(): number {
        return this.width * this.height;
    }
}

class Circle implements Shape {
    public radius: number;

    constructor(radius: number) {
        this.radius = radius;
    }

    public calculateArea(): number {
        return Math.PI * (this.radius * this.radius);
    }
}

class AreaCalculator {
    public calculateArea(shape: Shape): number {
        return shape.calculateArea();
    }
}

// TEST
// const rectangle = new Rectangle(3, 2);
// const circle = new Circle(12);
// const areaCalculator = new AreaCalculator();
// const test1 = areaCalculator.calculateArea(rectangle);
// const test2 = areaCalculator.calculateArea(circle);
```

이런 식으로 AreaCalculator 클래스를 인수를 갖는 하나의 함수로 단순화할 수 있고,
이 인수는 Shape 인터페이스를 기반으로 합니다.

## 3. Liskov Substitution Principle(LSP) - 리스코프 치환 원칙
### "부모 클래스에서 파생된 자식 클래스의 기능을 사용할 수 있어야 한다."
즉, 부모 클래스의 인스턴스 대신에 자식 클래스의 인스턴스로 대체해도 프로그램의 의미는 변화되지 않습니다. 부모 클래스와 자식 클래스의 행위는 일관되야 합니다.

아래 나쁜 예제가 있습니다. 정사각형(Square) 클래스는 직사각형(Rectangle) 클래스를 확장합니다.    
하지만 이 확장은 너비와 높이 속성을 덮어써서 로직이 변경되었기 때문에 의미가 없습니다.

```typescript
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  public _width: number;
  public _height: number;

  constructor(width: number, height: number) {
    super(width, height);

    this._width = width;
    this._height = height;
  }
}
```

그래서 덮어쓰는 것이 아닌, 정사각형(Square) 클래스를 제거하고 목적을 변경하지 않고 정사각형이라는 논리를 추가해줍니다. 

```typescript

class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }

  public isSquare(): boolean {
    return this.width === this.height;
  }
}
```

## 4. Interface Segregation Principle(ISP) - 인터페이스 분리 원칙
### "여러 개의 클라이언트별 인터페이스가 하나의 범용 인터페이스 보다 낫다"
즉, 인터페이스가 적은 것 보다는 많은 것이 좋다는 것입니다. 

```Character``` 라는 인터페이스를 구현하는 ```troll``` 이라는 클래스가 있습니다.    
하지만 트롤(troll) 은 수영과 말을 못합니다. 따라서 아래의 ```Character``` 인터페이스는 트롤(troll) 클래스에 적합하지는 않습니다.  

```typescript
interface Character {
  shoot(): void;
  swim(): void;
  talk(): void;
  dance(): void;
}

class Troll implements Character {
  public shoot(): void {
    // ...
  }
  
  public swim(): void {
    // a troll can't swim
  }

  public talk(): void {
    // a troll can't talk
  }

  public dance(): void {
    // ...
  }
}
```

**인터페이스 분리 원칙**에 따른다면 어떻게 바꿀 수 있을까요?
```Character``` 인터페이스를 제거하고 기능별 4개의 인터페이스로 분할하고 트롤(troll) 이 필요로 하는 인터페이스만 Troll 클래스를 의존하도록 변경합니다.

```typescript
interface Talker {
  talk(): void;
}

interface Shooter {
  shoot(): void;
}

interface Swimmer {
  swim(): void;
}

interface Dancer {
  dance(): void;
}

class Troll implements Shooter, Dancer {
  public shoot(): void {
    // ...
  }

  public dance(): void {
    // ...
  }
}
```

## 5. Dependency Inversion Principle(DIP) - 의존관계 역전 원칙
### "추상화에 의존해야지 구체화에 의존하면 안된다"
아래 코드는 나쁜 예제입니다. FrontendDeveloper 및 BackendDeveloper 클래스를 초기화하는 Software 클래스가 있습니다.
SoftwareProject 클래스는 FrontendDeveloper 와 BackendDeveloper 2 개의 클래스에 의존적입니다.
하지만 FrontendDeveloper 와 BackendDeveloper 는 비슷한 기능을 가져 비슷한 일을 하기 때문에 아래의 예제는 잘못된 방법입니다.    
의존관계 역전 원칙(DIP)의 목표를 달성하기 위한 요구 사항을 충족하는 더 좋은 방법이 있습니다.

```typescript
class FrontendDeveloper {
  public writeHtmlCode(): void {
    // ...
  }
}

class BackendDeveloper {
  public writeTypeScriptCode(): void {
    // ...
  }
}

class SoftwareProject {
  public frontendDeveloper: FrontendDeveloper;
  public backendDeveloper: BackendDeveloper;

  constructor() {
    this.frontendDeveloper = new FrontendDeveloper();
    this.backendDeveloper = new BackendDeveloper();
  }

  public createProject(): void {
    this.frontendDeveloper.writeHtmlCode();
    this.backendDeveloper.writeTypeScriptCode();
  }
}
```

우선 FrontendDeveloper 와 BackendDeveloper 는 유사한 클래스이므로 Developer 라는 인터페이스에 의존하게 합니다.    
SoftwareProject 클래스 내에서 FrontendDeveloper, BackendDeveloper 클래스를 초기화하는 것 대신에, 각 ```develop()``` 메서드를 호출하기 위해 반복하는 리스트(배열)로 사용합니다.    
즉, front, backend Developer 가 아니더라도 infra Developer 가 추가되는 등 클래스가 유연해집니다.

```typescript
interface Developer {
  develop(): void;
}

class FrontendDeveloper implements Developer {
  public develop(): void {
    this.writeHtmlCode();
  }

  private writeHtmlCode(): void {
    // ...
  }
}

class BackendDeveloper implements Developer {
  public develop(): void {
    this.writeTypeScriptCode();
  }

  private writeTypeScriptCode(): void {
    // ...
  }
}

class SoftwareProject {
  public developers: Developer[];

  public createProject(): void {
    this.developers.forEach((developer: Developer) => {
      developer.develop();
    });
  }
}
```

아직은 많이 부족하지만 위의 원칙을 계속 생각해나가면서 개발을 한다면 좋은 구조와 기능들을 지닌 코드들을 작성할 수 있을 것 같습니다!

### 출처
https://blog.bitsrc.io/solid-principles-in-typescript-153e6923ffdb