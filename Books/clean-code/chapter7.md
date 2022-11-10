# 7장 - 오류 처리

오류처리는 프로그램에 반드시 필요한 요소 중 하나이다. 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다.

## 오류 코드보다 예외를 사용하라

예외를 사용하면 논리가 오류 처리 코드와 뒤섞이지 않기 때문에 코드가 더욱 더 깔끔해지고, 품질도 나아진다.

## Try-Catch-Finally 문부터 작성하라

try 블록에서 어떤 일이 발생하든 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다. 그러므로 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 것이 좋다.

그러면 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

개발할 때는 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

## 미확인(unchecked) 예외를 사용하라

- unchecked exception: RuntimeException 의 하위 클래스로 실행 중에 발생할 수 있는 예외

확인(checked)된 예외는 OCP 를 위반한다. 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다. 대규모 시스템에서는 최상위 함수가 아래 함수를 호출하는 방식이 반복된다. 단계를 내려갈수록 호출하는 함수의 수는 늘어난다.

최하위 함수를 변경해 새로운 오류를 던진다고 가정해보자. 확인된 오류를 던진다면 함수는 선언부에 throws 절을 추가해야 한다. 그렇게 된다면 변경한 함수를 호출하는 함수 모두가 catch 블록에서 새로운 예외를 처리하거나, 선언부에 throw 절을 추가해야 한다는 말이다. 결과적으로 **최하위 단계에서 최상위 단계까지 연쇄적인 수정**이 발생한다.

때로는 확인된 예외도 유용하지만 일반적인 어플리케이션은 의존성이라는 비용이 더 크다.

## 예외에 의미를 제공하라

오류 메시지에 정보를 담아 예외와 함께 던져야 한다. 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨줘야 한다.

## 호출자를 고려해 예외 클래스를 정의하라

어플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 오류를 잡아내는 방법이 되어야 한다.

예외 클래스에 포함된 정보로 오류를 구분해도 괜찮은 경우에는 모든 예외를 정의하기 보다는 예외 클래스가 하나만 있어도 충분한 코드가 많다. 한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우라면 여러 예외 클래스를 사용한다.

```java
LocalPort port = new LocalPort(12);
try{
  port.open();
  } catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(),e);
  } finally {
  ...
  }
  
  
public class LocalPort {
  private ACMEPort innerPort;
  
  public LocalPort(int portNumber){
    innerPort = new ACMEPort(portNumber);
    }
    
  public void open() {
    try{
      innerPort.open();
      } catch(DeviceResponseException e) {
        throw new PortDeviceFailure(e);
        } catch(ATM1212UnlockedException e) {
          throw new portDeviceFailure(e);
        } catch (GMXError e) {
          throw new ProtDeviceFailure(e);
        }
       }
       ...
     }
```

위의 예제는 LocalPort 클래스는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 감싸기(Wrapper) 클래스이다. 외부 API 를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어들고, 다른 라이브러리로 갈아타더라도 비용이 적어진다.

## 정상 흐름을 정의하라

외부 API 를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리하는 것은 멋진 처리방식이지만, 때로는 중단이 적합하지 않는 경우도 있다.

```java
try {
	MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
	m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
	m_total += getMealPerDiem();
}
```

식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다. 식비를 비용으로 청구하지 않았다면 일일 기본 식비를 총계에 더한다. 그런데 예외가 논리를 따라가기 어렵게 만든다. 특수 상황을 처리할 필요가 없다면 더 좋은 코드가 나올 것이다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

위처럼 간결한 코드가 가능할까? ExpenseReportDAO 를 고쳐서 언제나 MealExpense 객체를 반환하게 하면 된다. 청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환하는 것이다.

```java
public class PerDiemMealExpenses implements MealExpenses {
	public int getTotal() {
		// 기본 값으로 일일 기본 식비를 반환한다.
	}
}
```

이를 **특수 사례 패턴(Special Case Pattern)**이라고 한다. 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.

이렇게 되면 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 왜냐하면 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하기 때문이다.

## null 을 반환하지 마라

한줄 건너 하나씩 null 을 확인하는 코드로 가득한 어플리케이션이 많다.

```java
public void registerItem(Item item) {
	if (item != null) {
		ItemRegistry registry = peristentStore.getItemRegistry();
		if (registry != null){
			//...
		}
	}
}
```

null 을 반환하는 **코드는 처리할 일을 늘릴 뿐만 아니라 호출자에게 문제를 떠넘기는 것**이다. 누구 하나라도 null 확인을 빼먹는다면 어플리케이션이 통제 불능에 빠질지도 모른다.

만약 메서드에서 null 을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환헤야 한다.

많은 경우가 특수 사례 객체가 손쉬운 해결책이다.

```java
List<Employee> employees = getEmployees();
if (employees != null) {
	for (Employee e: employees) {
		totalPay += e.getPay();
	}
}
```

위와 같은 경우 반드시 null 을 반환한 필요가 없다. 빈 리스트를 반환한다면 오히려 코드가 훨씬 깔끔해진다.

```java
List<Employee> employees = getEmployees();
for (Employee e: employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if (직원이 없다면...) {
		return Collections.emptyList();
	}
}
```

## null 을 전달하지 마라

메서드에서 null 을 반환하는 방식도 나쁘지만 메서드로 null 을 전달하는 방식은 더 나쁘다. 정상적인 인수로 null 을 기대하는 API 가 아니라면 메서드로 null 을 전달하는 코드는 최대한 피해야 한다.

두 지점 사이의 거리를 계산하는 메서드가 있다.

```java
public class MetricsCalculator {
	public double xProjection(Point p1, Point p2) {
		return (p2.x - p1.x) * 1.5;
	}
	//...
}
```

누군가가 null 을 전달하면 어떻게 될까?

```java
calculator.xProjection(null, new Point(12, 13));
```

`NullPointerException` 이 발생한다.  새로운 예외 유형을 만들어 던지는 방법과 assert 문을 사용하는 등, 해결할 수 있는 방법들이 있다. 대다수의 프로그래밍 언어는 호출자가 실수로 넘기는 null 을 적절하게 처리할 수 있는 방법이 없다. 즉, 인수로 null 이 넘어오면 코드에 문제가 있다고 봐도 된다.

깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. **오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려**하면 튼튼하고 깨끗한 코드를 작성할 수 있다. 또한 코드의 유지보수성도 크게 높아진다.
