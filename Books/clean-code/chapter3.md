# 3장

### 작게 만들어라

함수를 만드는 첫 번째 규칙은 작게이다. 필자의 경험 상 작은 함수가 좋다고 한다.

if/else, while 문 등에 들어가는 블록은 한 줄이어야 한다. 즉, 중첩구조가 생길 만큼 함수가 커지면 안된다는 것이다. 그래야 함수를 읽고 이해하기 쉬워진다.

### 한 가지만 해라

함수는 한 가지의 역할만 잘 수행하도록 짜야 한다.

함수가 한 가지만 하는 지 판단하는 방법이 있다. 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 것이다.

### 함수 당 추상화 수준을 하나로

함수가 한 가지만 처리하기 위해서는 함수 내 모든 문장의 추상화 수준이 동일해야 한다.

어떤 문단을 읽듯이 프로그램이 읽혀야 한다.

각 함수는 다음 함수를 소개하는 방식으로 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 와야 한다.

핵심은 함수는 짧으면서 한 가지만 하는 함수를 만들어야 된다는 것이다.

### Switch 문

Switch 문을 추상 팩토리에 꽁꽁 숨겨서 아무에게도 구현부분을 보여주지 않는다.

```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay();
}
-----
public interface EmployeeFactory {
	public Employee makeEmployee (EmployeeRecord r) throws InvalidEmployeeType;
}
-----
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee (EmployeeRecord r) throws InvalidEmployeeType {
		switch(r.type) {
			case COMMISSTIONED;
				return new CommisionedEmployee(r);
			//...
			default:
				throw new InvalidEmployeeType(r.type)
		}
	}
}
```

필자는 다형적 객체를 생성하는 코드에 한해서 Switch 문을 한 번정도는 허용해준다고 한다.

### 서술적인 이름을 사용하라

이름은 길어도 괜찮다. 오히려 짧고 어려운 이름보다 **길고 서술적인 이름**이 더 좋다.

이름을 붙일 때는 일광성이 있어야 한다. 모듈 내에서 함수 이름은 같은 문구, 명사, 동사를 사용한다.

```
includeSetupAndTeadownPages, includeSetupPages, includeSuiteSetupPages,...
```

### 함수 인수

함수에서 이상적인 인수 개수는 0~2 개까지 이다. 3 개는 가능한 피하면 좋고, 4 개 이상은 특별한 이유가 필요하다.

> 단항 함수
>

함수에 인수 1개를 넘기는 이유로 가장 흔한 경우는 2 가지가 있다.

첫 번째로는 인수에게 질문을 던지는 경우이다.  boolean fileExists(”Myfile”) 이 좋은 예이다.

두 번째로는 인수를 뭔가로 변환해 결과를 반환하는 경우이다.

다소 드물게 사용하는 경우는 단항 함수 형식 이벤트이다. 입력 인수만 있고 출력 인수는 없는 것으로 이벤트로 해석해 입력 인수로 시스템 상태를 바꾼다. 이벤트 함수는 이벤트라는 사실이 코드에 명확히 드러나야 한다.

플래그 인수를 넘기는 것은 끔찍한 짓이다. 왜냐하면 함수가 한꺼번에 여러 가지를 처리한다고 대놓고 표현하는 것이기 때문이다(함수는 한가지 일을 잘 수행하는 것이 중요하다).

> 이항 함수
>

기본적으로 인수가 2개인 함수는 인수가 1 개인 함수보다 이해하기 어렵다.

이항 함수의 좋은 예로는 대표적으로 `Point P = new Point(0, 0)` 이다. 직교 좌표계 점은 일반적으로 인수 2개를 취하기 때문이다.

그리고 2 개의 인수 타입이 같다면 실수로 인수 값을 잘못 넣는 실수도 종종 일어난다.

이함 함수가 무조건 나쁜 것은 아니지만 위험이 따르기 때문에 최대한 단항 함수로 바꾸도록 애써야 한다.

> 삼항 함수
>

인수가 3 개라면 인수가 2개인 함수보다 훨씬 더 이해하기 어렵다.

만얀 인수가 2-3 개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어봐야 한다.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Pointer center, double radius);
```

객쳋를 생성해 인수를 줄이는 것이다.

함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름은 필수다.

`write(name)` 보다는 `writeField(name)` 처럼 말이다.

그리고 함수명에 키워드를 추가하는 방식을 통해 인수 순서를 기억할 필요 없도록 만들어줄 수도 있다.

예를 들어 `assertEquals` 보다 `assertExpectedEqualsActual` 이 더 좋다.

### 부수 효과를 일으키지 마라

```java
public boolean checkPassword(String name, String password) {
		// Check Logic
		if ("Valid Password".equals(hasedPassword)) {
				Session.initialize();
				//...
		}
} 
```

위의 코드에서 부수 효과를 일으키는 것은 `Session.initialize();`   이다. 함수명을 봤을 때는 세션을 초기화 한다는 내용이 들어가 있지 않는다. 그래서 함수 이름만 보고 함수를 호출하는 개발자는 사용자 인증을 하면서 기존 세션 정보가 지워지는 상황에 처하게 된다.

따라서 `checkPassword` 보다는 `checkPasswordAndInitializeSession` 이라는 이름이 훨씬 더 좋다.

> 출력인수
>

일반적으로 출력 인수는 피해야 한다. 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택한다.

### 명령과 조회를 분리하라!

함수는 뭔가를 수행하거나 답하거나 둘 중에 하나만 해야 한다. 객체 상태를 변경하거나 아니면 객체 정보를 반환하거나 둘 중 하나다.

```java
public boolean set(String attribute, String value);

if(set("userName", "unclebob")) ...
```

위의 코드는 userName 이 unclebob 으로 설정하는지, 확인하는 코드인지 의미가 모호하다.

setAndCheckIfExists 라고 바꾸는 방법도 있지만 명령과 조회를 분리하는 것이 더 좋다.

```java
if(attribute("userName")) {
	setAttribute("userName", "unclebob");
	...
}
```

### 오류 코드보다 예외를 사용하라!

오류 코드 대신 예외를 사용하면 오류 처리가 원래 코드에서 분리되므로 코드가 깔끔해진다.

```java
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey) == E_OK) {
				//...
		} else {
				//...
		}
	} else {
			//...
	}
} else {
	return E_ERROR;
}

// exception
try {
	deletePage(page);
	registry.deleteReference(page.name)
	configKeys.deleteKey(page.name.makeKey());
} catch (e) {
	logger.log(e.getMessage());
}
```

> Try/Catch 블록 뽑아내기
>

try-catch 블록은 코드 구조에 혼란을 일으키며, 정상 동작과 오류 처리 동작을 뒤섞기 때문에 별도의 함수로 뽑아내는 편이 좋다.

```java
private void deletePageAndAllReferences(Page page) throw Exception {
	deletePage(page);
	registry.deleteReference(page.name)
	configKeys.deleteKey(page.name.makeKey());
}

public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
	} catch (Exception e) {
		logError(e);
	}
}
```

위의 `delete` 함수는 모든 오류를 처리하기 때문에 코드를 이해하기 쉬워진다. 실제로 기능을 수행하는 것은 `deletePageAndAllReferences` 함수이다. 이렇게 정상 동장과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.

> 오류 처리도 한 가지 작업이다
>
