# 4장

우리는 코드로 의도를 제대로 표현하지 못하기 때문에, 즉 실패를 만회하기 위해 주석을 사용한다.

그러므로 주석이 필요한 상황이 온다면 코드로 의도를 표현할 수는 없는지 다시 생각해봐야 한다.

부정확한 주석은 아예 없는 주석보다 훨씬 더 나쁘다. 독자를 현혹하고 헷갈리게 한다. 진실은 코드에만 존재한다. 코드만이 자신이 무엇을 하는지 말하고 있고, 정확한 정보를 제공하는 유일한 출처이다.

### 주석은 나쁜 코드를 보완하지 못한다

일반적으로 코드에 주석을 추가하는 것은 나쁜 코드라는 것이다. 주석으로 설명할 시간에 코드를 통해 풀어나가는게 더 좋은 방법이다.

### 코드로 의도를 표현해라

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))

------

if (employee.isEligibleForFullBenefits())
```

많은 경우 주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다.

### 좋은 주석

필요한 주석들도 존재한다.

> 법적인 주석
>

저작권 정보와 소유권 정보는 필요하고 타당하다. 그 외에 조항과 조건을 열거하는 대신에, 가능하다면, 표준 라이선스나 외부 문서를 참조해도 된다.

> 정보를 제공하는 주석
>

기본적인 정보를 주석으로 제공하면 편리하다.

```java
// 테스트 중인 Responder 인스턴스를 반환한다.
protected abstract Responder responderInstance();
```

하지만 가능하다면 함수 이름에 정보를 담는 편이 더 좋다.

```java
protected abstract Responder responderBeingTested();
```

> 의도를 설명하는 주석
>

때로는 주석은 구현을 이해하게 도와주는 선을 넘어 결정에 깔린 의도까지 설명한다.

```java
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다
for (int i = 0; i < 25000; i++) {
	WidgetBuilderThread widgetBuilderThread = 
		new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
	Thread thread = new Thread(widgetBuilderThread);
	thread.start();
}
```

> 의미를 명료하게 밝히는 주석
>

인수나 반환 값이 표준 라이브러리나 변경하지 못하는 코드에 속한다면 의미를 명료하게 밝히는 주석이 유용하다.

```java
assertTrue(a.compateTo(a) == 0);  // a == a
assertTrue(a.compateTo(b) != 0);  // a != b
assertTrue(a.compateTo(ab) == 0);  // a == b
assertTrue(a.compateTo(b) == -1);  // a < b
```

위와 같은 주석을 달 때는 더 나은 방법이 없는지 고민하고 정확히 달아야 한다.

> 결과를 경고하는 주석
>

다른 프로그래머에서 결과를 경고할 목적으로 주석을 사용한다.

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오
public void _testWithReallyBigFile() {
	//...
}
```

> TODO 주석
>

앞으로 할 일을 `//TODO` 주석으로 남겨두면 편하다.

```java
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
	return null;
}
```

TODO 주석을 통해 필요하지만 당장 구현하기 어려운 업무를 기술한다.

> 중요성을 강조하는 주석
>

중요하지 않다고 생각할 수 있는 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

```java
String listItemContent = match.group(3).trim();
// 여기서 trim 은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
```

> 공개 API 에서 Javadocs
>

설명이 잘 된 Open API 는 참으로 유용하고 만족스럽다.

### 나쁜 주석

대다수가 나쁜 주석에 속한다. 일반적으로 대다수 주석은 허출한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.

> 주절거리는 주석
>

답을 알기 위해 **다른 코드들을 뒤져봐야 하는 주석**은 독자와 제대로 소통하지 못하는 주석이다. 그런 주석은 바이트만 낭비할 뿐이다.

> 같은 이야기를 중복하는 주석
>

주석이 코드의 내용을 그래도 중복하게 된다면 코드보다 주석을 읽는 시간이 더 오래 걸릴 수가 있다.

```java
// this.closed 가 true 일 때 반환되는 유틸리티 메서드이다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis) throw Exception {
	if (!closed) {
		wait(timeoutMillis);
		if (!closed) {
			throw new Exception("exception");
		}
	}
}
```

위 코드는 주석이 제공하는 내용이 실제로 코드보다 부정확해 독자가 함수를 대충 이해하고 넘어가게 만든다. 엔진 후드를 열어볼 필요가 없다고 고객한테 회유하는 중고차 판매원과 비슷한 것이다.

> 오해할 여지가 있는 주석
>

주석에 담긴 살짝 잘못된 정보로 인해 프로그래머가 경솔하게 함수를 호출할지도 모르는 상황이 발생한다.

그렇게 되면 프로그래머는 어떻게 돌아가는지에 대한 이유를 찾느라 힘든 상황이 생길지도 모른다.

> 의무적으로 다는 주석
>

```java
/**
*
* @param title CD 제목
* @param author CD 저자
*/
public void addCD(String title, String author) {
	CD cd = new CD();
	cd.title = title;
	cd.author = author;
}
```

위의 예제와 같이 아무 가치 없는 주석은 코드만 헷갈리게 만들며, 거짓말할 가능성을 높이며, 잘못된 정보를 제공할 여지만 만든다.

> 이력을 기록하는 주석
>
