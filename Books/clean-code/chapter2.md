# 2 장 - 의미있는 이름

변수, 함수, 클래스 등 네이밍을 할 때 의도를 분명하게 밝히는 것이 중요하다. 만약 주석이 필요하다면 올바른 네이밍을 사용하지 못한 것이다.

불용어(의미없는 단어) 를 추가한 이름은 아무런 정보도 제공하지 못한다. 다른 클래스를 ProductInfo 혹은 ProductData 라고 부른다면 개념을 구분하지 않은 채 이름만 다르게 한 경우이다. Info, Data는 a, an, the 와 마찬가지로 의미가 불분명한 불용어이다.

```tsx
// 팀 내 개발자는 아래 문구중 어떤 것을 호출해야 할지 모르게 된다.
getActiveAccount();
getActiveAccounts();
getActiveAccountInfo();
```

상수에 버그가 생긴다면 검색으로 찾아서 해결하기 매우 힘들기 때문에 변수로 만들어서 사용하는 것이 좋다.

```tsx
for (int j = 0; j < 34; j++) {
  s += (t[j]*4)/5;
}

int realDaysPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j = 0; j < NUMBER_OF_TASKS; j++) {
  int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
	int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK)
	sum += realTaskWeeks;
}
```

인터페이스 클래스의 이름에 `IXxx` 이런식으로 붙이는 것은 주의를 흐트리고 과도한 정보를 제공하는 것이다. 따라서 인터페이스 클래스 이름과 구현 클래스 이름 중 하나를 인코딩해야 된다면 구현 클래스 이름을 인코딩하는 것이 좋다.

```
Interface: IShapeFactory -> ShapeFactory
Use-Class: ShapeFactory -> ShapeFactoryImpl
```

### 자신의 기억력을 자랑하지 마라

전문가 프로그래머는 자신의 능력을 좋은 방향으로 사용해 남들이 이해하는 코드를 내놓는다.

### 클래스/메서드 이름

클래스 이름은 명사나 명사구가 적합하고, 메서드 이름은 동사나 동사구가 적합하다.

### 한 개념에 한 단어를 사용하라

일관성 있는 어휘를 사용하는 것이 좋다.

### 말 장난을 하지 마라

한 단어를 2 가지 이상의 목적으로 사용하면 안된다. 예를 들면 add 라는 메서드는 기존 값 두 개를 더하거나 이어서 새로운 값을 만든다고 가정해보자. 근데 집합에 값을 추가시키는 기능의 메서드명을 add 라고 불러도 되는 것인가? 기존 add 메서드와 맥락이 다르기 때문에 Insert 나 append 라는 이름이 더 적당하다.

### 해법 영역에서 가져온 이름을 사용하라

전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등은 사용해도 괜찮다.

기술 개념에는 기술 이름이 가장 적합한 선택이다.

### 의미 있는 맥락을 추가하라

스스로 의미가 분명한 이름이 없지 않다. 하지만 대다수 이름은 그렇지 못한다.

그래서 클래스, 함수, 이름 공간에 넣어 맥락을 부여하고, 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.

단순 state 라는 변수를 사용하기 보다는 addr 이라는 접두어를 추가해 addrFirstName, addrLastName, addrState 라 쓰면 맥락이 좀 더 분명해진다.

### 불필요한 맥락을 없애라

의미가 분명한 경우에 한해서 일반적으로 짧은 이름이 긴 이름보다 좋다. 이름에 불필요한 맥악을 추가하지 않도록 주의해야 한다.
