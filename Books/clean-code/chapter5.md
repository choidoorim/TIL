# 5장 - 형식 맞추기

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야 한다. 팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다. 필요하다면 규칙을 자동으로 적용하는 도구를 활용해야 한다.

## 형식을 맞추는 목적

돌아가는 코드가 전문 개발자의 일차적인 의무라고 생각할 수도 있지만, 저자는 생각이 바뀌길 바란다.

오늘 구현한 기능이 다음 버전에서 바뀔 확률은 상당히 높다. 그런데 오늘 구현한 코드의 가독성은 **앞으로 바뀔 코드의 품질에 지대한 영향**을 미친다. 처음에 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다.

## 적절한 행 길이를 유지하라

JUnit, FitNesse, testNG, tam, Ant, Tomcat 프로젝트를 조사했을 때 평귤 파일의 행(줄) 길이는 약 65 줄이다.

500 줄을 넘어가는 파일이 없으며 **대다수가 200 줄 미만**이다. 즉, 500 줄을 넘지 않고 대부분 200 줄 정도인 파일로도 커다란 시스템을 구축할 수 있다는 뜻이다. 반드시 지킬 필요는 없지만 바람직한 규칙이다. 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽기 때문이다.

### 신문 기사처럼 작성하라

첫 문단은 전체 기사 내용을 요약하고, 세세한 사실은 숨기고 큰 그림을 보여준다. 쭉 읽으며 내려가면 세세한 사실이 조금씩 드러난다. 날짜, 이름, 발언, 주장, 기타 세부사항이 나온다.

소스 파일, 코드도 신문기사와 비슷하게 작성해야 한다. 이름은 간단하면서도 설명이 가능할 정도로 신경써서 지어야 한다. 아래로 내려갈수록 의도를 세세하게 묘사해야 한다.

### 개념은 빈 행으로 분리하라

개념마다 빈 행을 넣어 분리해야 한다.

```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "''' .+?'''";
	
	public BoldWidget(ParentWidget parent) throws Exception {
		super(parent);
	}
}
```

만약 빈행을 넣지 않는다면 코드 가독성이 매우 나뻐지고, 암호처럼 보이게 된다.

```java
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "''' .+?'''";
	public BoldWidget(ParentWidget parent) throws Exception {
		super(parent);
	}
}
```

### 세로 밀집도

줄바꿈이 개념을 뜻한다면 세로 밀집도는 연관성을 의미한다.

즉, 서로 밀접한 코드 행은 세로로 가까이 놓여야 된다는 것이다.

```java
public class ReporterConfig {
	private String m_className;
	private List<Property> m_properties = new ArrayList<Property>();
	
	public void addProperty(Property property) {
```

### 수직거리

같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성(한 개념을 이해하는데 다른 개념이 중요한 정도)을 표현한다. 만약 연관성이 깊은 두 개념이 멀리 떨어져 있으면 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 찾게 된다.

## 변수 선언

변수는 사용하는 위치에 최대한 가까이 선언한다.

## 인스턴스 변수

인스턴스 변수는 클래스 맨 처음에 선언한다. 자바에서는 보통 클래스 맨 처음에 인스턴스 변수를 선언한다.

## 종속함수

하나의 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 또한 가능하다면 **호출하는 함수를 호출되는 함수보다 먼저 배치**한다. 그러면 프로그램이 자연스럽게 읽기 쉬워진다.

```java
public Response makeResponse() {
	String pageName = getPageNameOrDefault(request, "FrontPage");
}

private String getPageNameOrDefault(Request request, String defaultPageName) {
//...
```

## 개념적 유사성
