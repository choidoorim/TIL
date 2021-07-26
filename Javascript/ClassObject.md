### :clipboard: Class 와 Object

#### Class
- 연관 있는 데이터들이 fields 와 method 로 종합적으로 묶여있는 것
  - Data Class : fields 로만 이루어진 Class
- Class 는 데이터가 직접 들어가 있지는 않으며, 들어올 수 있는 데이터를 한 번만 선언한다. 붕어빵을 만드는 틀과 비슷하다고 생각하면 된다


#### Object
- Class 에서 실제로 데이터를 넣어 만들어진 것으로 Class 에서 새로운 instance 를 생성하면 Object 가 되는 것이다
- Class 를 이용해서 여러 개의 Object 생성이 가능하다
- Class 는 틀만 있는 것이기에 메모리에 올라가 있지 않지만, Object 는 데이터를 넣기 때문에 메모리에 올라가 있다
- Class 가 붕어빵을 만드는 틀이라면 Object 는 틀로 만든 팥, 크림 붕어빵에 비유할 수 있다
- constructor : 생성자로 클래스로부터 객체를 생성할 때 호출되어 객체의 초기화를 담당한다

:bookmark_tabs: Class, Object Example Code
```javascript
class person {
    constructor(name, age) {    // constructor - 생성자
        this.name = name    // 전달받은 데이터를 name 과 age 에 할당한다
        this.age = age
    }

    speak() {
        console.log(`${this.name}: Hello!`)
    }
}

const alvin = new person('alvin', 25)
console.log(alvin.name)
console.log(alvin.age)
```

#### Getter and Setter
- Getter : ' this.변수 = 변수 ' 선언 시 ' this.변수 '는 메모리에 있는 변수를 불러오는 것이 아닌 getter 를 호출한다
- Setter : ' this.변수 = 변수 ' 선언 시 ' = 변수 '는 메모리에 변수를 할당하는 것이 아닌 setter 를 호출한다
  - Setter 선언 시 주의할 점 : 아래와 같이 setter 명과 this.변수명이 같을 경우 this.age 가 set age()를 무한정 호출하는 Error(Call stack) 가 발생할 수 있다
  ```
  set age(value) { 
        this.age = value;
  }
  ```
  따라서 아래와 같이 ' _ ' 을 추가하여 Error 를 해결할 수 있다
  ```
  set age(value) { 
        this._age = value;
  }
  ``` 

:bookmark_tabs: Getter and Setter Example Code
- user Class 에는 firstName, lastName, _age 라는 총 3개의 field 가 있다
```javascript
class user {
    constructor(firstName, lastName, age) {
        this.firstName = firstName
        this.lastName = lastName
        this.age = age
    }

    get age() {
        return this._age;
    }

    set age(value) {
        if (value < 0) {
            throw Error('age can not be negative')
        }
        this._age = value;
    }
}

const user1 = new user('choi', 'alvin', -1)
console.log(user1.age)
```

#### public and private
- public 은 class 내 외부에서 활용할 수 있지만, private 는 class 내부에서만 활용이 가능하고 외부에서는 접근할 수 없다
- private field 는 해쉬 기호(#)를 붙이면 된다 

:bookmark_tabs: public and private Example Code
```javascript
class Experiment {  // ' # ' 해쉬기호를 붙이면 private field 이다. class 내부에서만 값을 활용할 수 있고 외부에서는 사용할 수 없다 
    publicField = 2;
    #privateField = 0;
}
const experiment = new Experiment()
console.log(experiment.publicField)   // 2
console.log(experiment.privateField)  // undefined
```

#### Static 
- Object 에 상관 없이 Class 에서 공통적으로 사용할 수 있는 것이라면 static 을 사용하면 된다 

:bookmark_tabs: static Example Code
```javascript
class Article {
    static publisher = 'Alvin';
    constructor(articleNum) {
        this.articleNum = articleNum;
    }

    static printPublisher() {
        console.log(Article.publisher)
    }
}

const article1 = new Article(1);
const article2 = new Article(2);
console.log(article1.publisher) // undefined
console.log(Article.publisher)  // Alvin
Article.printPublisher()        // Alvin
```

#### Inheritance - 상속
- extends 라는 키워드를 이용해서 class 의 fields 나 method 를 연장해서 사용할 수 있다
- 상속 받은 클래스에서 동일한 메소드는 재정의해서 사용할 수 있는데, 이것을 바로 오버라이딩이라고 한다
- ' Super.메소드명 '을 사용하면 상속받은 부모의 메소드(기능)도 호출되고, 기존 메소드를 변경할 수도 있다

:bookmark_tabs: Inheritance, Overriding Example Code
```javascript
class shape {
    constructor(width, height, color) {
        this.width = width
        this.height = height
        this.color = color
    }

    draw() {
        console.log(`drawing ${this.color} color of`)
    }

    getArea() {
        return this.width * this.height
    }
}

class Rectangle extends shape {}
class Triangle extends shape {
    draw() {
        super.draw(); // super : 부모 method의 draw 기능을 호출
        console.log('Triangle');
    }
    getArea() {   // overriding : 부모 메소드의 기능을 재정의(변경)하여 사용
        return (this.width * this.height) / 2;
    }
}

const rectangle = new Rectangle(10, 10, 'red')
rectangle.draw()
console.log(rectangle.getArea())
const triangle = new Triangle(10, 10, 'blue')
triangle.draw()
console.log(triangle.getArea())
```