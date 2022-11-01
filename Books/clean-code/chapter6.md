# 6장 - 객체와 자료구조

변수를 private 로 정의하는 이유는 남들이 변수에 의존하지 않게 만들고 싶기 때문이다.

## 자료 추상화

```java
// 구현을 외부에 노출하는 구체적인 클래스
public class Point {
	public double x;
	public double y;
}

// 구현을 완전히 숨기는 추상적인 클래스
public interface Point {
	double getX();
	double getY();
	void setCartesian(double x, double y);
	double getR();
}
```

추상적인 인터페이스 클래스는 자료구조를 명백히 표현한다. 또한 클래스 메서드가 접근 정책을 강제하고 있다.

구체적인 클래스는 구현을 노출하고 있다. 변수를 private 로 선언한다고 해도 각 값마다 조회 함수와 설정 함수를 제공하게 된다면 구현을 외부로 노출하는 셈이다.

변수 사이에 함수라는 계층을 추가하더라도 구현이 저절로 감춰지지는 않는다. 구현을 감추기 위해서는 **추상화**가 필요하다. 추상화 인터페이스를 제공해 **사용자가 구현을 모른 채 자료의 핵심을 조작**할 수 있어야 진정한 의미의 클래스이다.

```java
// 구체적인 클래스
public interface Vehicle {
	double getFuelTankCapacityInGallons();
	double getGallonsOfGasoline();
}

// 추상적인 클래스
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

그리고 **클래스의 정보들을 세세하게(구체적으로) 공개하기 보다는 추상적인 개념으로 표현하는 것이 좋다**. 또한 아무 생각 없이 Getter/Setter 함수를 추가하는 방법이 가장 나쁜 방법이다.

## 자료/객체 비대칭

객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다. 그리고 자료 구조는 자료를 그대로 공개하면 별다른 함수를 제공하지 않아야 한다.

```java
// 절차지향적 클래스
public class Square {
	public Point topLeft;
	public double side;
}

public class Rectangle {
	public Point topLeft;
	public double height;
	public double width;
}

public class Circle {
	public Point center;
	public double radius;
	public double width;
}

public class Geometry {
	public final double PI = 3.141592653585793;

	public double area(Object shape) throws NoSuchShapeException {
		if(shape instanceOf Square) {
			Square s = (Square)shape;
			return s.side * s.side;
		}
	else if(shape instanceOf Rectangle) {
			Rectangle r = (Rectangle)shape;
			return r.height * r.width;
		}
	else if(shape instanceOf Circle) {
			Circle c = (Circle)shape;
			return PI * c.radius * c.radius
		}
	}
}
```

각 도형 클래스(Square, Rectangle, Circle)는 간단한 자료구조이고, 도형이 동작하는 방식은 Geometry 클래스에서 구현한다. 도형 클래스에서는 아무 메서드도 제공하지 않는 것이다.

Geometry 클래스에 **함수를 추가한다면 도형 클래스들은 아무 영향을 받지 않지만 새 도형을 추가하고 싶다면 Geometry 클래스에 속한 함수를 모두 고쳐야 한다**.

```java
// 객체 지향적인 도형 클래스
public class Square implements Shape {
	public Point topLeft;
	public double side;

	public double area() {
		return side*side;
	}
}

public class Rectangle implements Shape {
	public Point topLeft;
	public double height;
	public double width;

	public double area() {
		return height*width;
	}
}

public class Circle implements Shape {
	public Point center;
	public double radius;
	public double width;

	public double area() {
		return PI*radius*radius;
	}
}
```

객체지향적인 클래스에서는 Geometry 클래스가 필요없다. 그러므로 새 도형을 추가해도 **기존 함수에는 아무런 영향을 미치지 않는다. 반면 새 함수를 추가하고 싶다면 도형 클래스를 전부 고쳐야 한다**.

절차지향적인 클래스와 객체지향적인 클래스는 상호 보완적인 특징이 있다.

즉, 객체지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체지향코드에서 쉽다. 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

## 디미터 법칙

잘 알려진 휴리스틱으로, 모듈은 자신이 조작하는 객체의 내부를 몰라야 한다는 법칙이다. 즉, 객체는 조회 함수로 내부 구조를 공개하면 안된다는 의미다.

디미터 법칙은 **“클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다”** 라고 주장한다.

```
- 클래스 C
- f가 생성한 객체
- f인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체
```

하지만 위 객체에서 허용된 메서드가 반환하는 객체의 메서드는 호출하면 안된다.

```java
// 디미터 법칙을 어기는 코드
final String ouputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

### 기차 충돌

위의 디미터 법칙을 어기는 예제 코드를 기차 충돌(train wreck) 이라고 부른다. 여러 객체가 한 줄로 이어진 기차처럼 보이기 때문이다. 일반적으로 조잡하기에 피하는 것이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String ouputDir = scratchDir.getAbsolutePath();
```

위와 같이 바꾸는 것이 좋다.

디미터 법칙을 위반하는지 여부는 ctxt, Options, ScratchDir 이 객체인지 자료구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 하지만, 자료구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

위와 같이 구현했다면 디미터 법칙을 거론할 필요가 없어진다.

### 잡종 구조

때로는 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.

이러한 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵기 때문에 되도록 피하는 편이 좋다.

### 구조체 감추기

만약 객체라면 **무언가를 하라고** 말해야지 속을 드러내라고 말하면 안된다.

## 자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스이다. 이런 자료 구조체를 때로는 자료 전달 객체(DTO - Data Transfer Object) 라 한다. 흔히 DTO 는 데이터베이스에 저장된 가공되지 않은 정보를 어플리케이션 코드에서 사용할 객체로 변환하는 단계에서 가장 처음으로 사용하는 구조체이다.

### 활성 레코드

DTO 의 특수한 형태이다. 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하면 좋지 않다. 자료 구조도 아니고, 객체도 아닌 잡종 구조가 나오기 때문이다.

활성 레코드는 자료 구조로 취급하여 해결해야 한다. 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다.

## 결론

객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.

자료구조는 동작없이 자료를 노출한다. 그래서 기존 자료구조에 새 동작을 추가하기는 쉽지만, 새 자료구조를 추가하기는 어렵다.
