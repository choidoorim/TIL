# 예외처리(exception handling)
## 1. 프로그램 오류
프로그램이 실행 중 어떤 원인에 의해서 오작동을 하거나 비정상적으로 종료되는 경우가 있습니다.
이러한 결과를 초래하는 원인을 프로그램 에러 또는 오류라고 합니다.

- 컴파일 에러: 컴파일 시에 발생하는 에러
- 런타임 에러: 실행 시에 발생하는 에러
- 논리적 에러: 실행은 되지만, 의도와 다르게 동작하는 것

자베에서는 실행 시(runtime) 발생할 수 있는 프로그램 오류를 **에러(error)** 와 **예외(exception)** 두 가지로 구분합니다.
에러는 메모리 부족이나 스택오버플로우와 같이 일단 발생하면 복구할 수 없는 심각한 오류이고, 예외는 발생하더라도 수습될 수 있는 비교적 덜 심각한 것 입니다.
아래 사진은 예외클래스의 계층도입니다.

![에러계층구조](https://user-images.githubusercontent.com/63203480/147412203-f3621b1e-4ade-4a28-9a66-728bc8bb6479.png)    
사진출처: https://tenlie10.tistory.com/27

- Exception Class: RuntimeException 을 제외한 Exception 클래스들로, 유저의 실수와 같은 외적인 요인에 의해 발생하는 예외
- RuntimeException Class: 개발자의 실수로 발생하는 예외

## 2. 예외처리하기 - try catch 문
프로그램의 실행도중에 발생하는 에러는 어쩔 수 없지만, 예외는 프로그래머가 이에 대한 처리를 미리 해줘야합니다.

```
예외처리(exception handling) 의
    정의 - 프로그램 실행 시 발생할 수 있는 예외에 대비한 코드를 작성하는 것.
    목적 - 프로그램의 비정상 종료를 막고, 정상적인 실행상태를 유지하는 것.
```

예외를 처리하기 위해서는 ```try-catch``` 문을 사용하며, 그 구조는 아래와 같습니다.

```java
try {
    // 예외가 발생할 가능성이 있는 문장들을 넣는다.
} catch(Exception1 e1) {
    // Exception1 이 발생했을 때, 이를 처리하기 위한 코드
} catch(Exception2 e2) {
    // Exception2 이 발생했을 때, 이를 처리하기 위한 코드     
}
```

하나의 try 블럭 다음에는 여러 종류의 예외를 처리할 수 있도록 하나 이상의 catch 블럭이 올 수 있습니다.
이 중 발생한 **예외의 종류와 일치하는 단 한 개의 catch 블럭만 수행**됩니다.


## 3. try-catch 문의 흐름
- try 블럭 내에서 예외가 발생한 경우,
    1. 발생한 예외와 일치하는 catch 블럭이 있는지 확인한다.
    2. 일치하는 catch 블럭을 찾게 되는 시점에 try 문을 실행하지 않고, 그 catch 블럭 내의 문장들을 수행하고 전체 try-catch 문을 빠져나가서
    그 다음 문장을 계속해서 수행한다. 만일 일치하는 catch 블럭을 찾지 못하면, 예외는 처리되지 못한다.
       
- try 블럭 내에서 예외가 발생하지 않은 경우
    1. catch 블럭을 거치지 않고 전체 try-catch 문을 빠져나가서 수행을 계속한다.
    
## 4. 예외의 발생과 catch 블럭
```catch``` 블럭은 괄호()와 블럭{} 두 부분으로 나눠져 있는데, 괄호에는 처리하고자 하는 예외와 같은 타입의 참조변수 하나를 선언해야 합니다.
일치하는 catch 블럭을 찾게 되면 블럭에 있는 문장들을 모두 수행한 후에 ```try-catch``` 문을 빠져나가고 예외처리 됩니다.

모든 예외 클래스는 Exception 클래스의 자손이므로, catch 블럭의 괄호()에 Exception 클래스 타입의 참조변수를 선언해 놓으면 어떤 종류의 예외가 발생하더라도
반드시 catch 블럭에 의해서 처리됩니다.
```java
//...
catch (Exception e) {
    //...
}
```

### printStackTrace()와 getMessage()
예외가 발생했을 때 생성되는 예외 클래스의 인스턴스에는 발생한 예외에 대한 정보가 담겨져 있으며, 
```getMessage()``` 와 ```printStackTrace()``` 를 통해서 이 정보를 얻을 수 있습니다. catch 블럭 내에서만 사용 가능합니다.

```java
try{
    //...   
}
catch (Exception e) {
    e.printStackTrace();
}
```

```printStackTrace(PrintStream s)``` 또는 ```printStackTrace(PrintWriter s)``` 를 사용하면 발생한 예외에 대한 정보를 파일에 저장할 수 있습니다.

### 멀티 catch 블럭
여러 catch 블럭을 '|' 기호를 통해, 하나의 catch 블럭으로 합칠 수 있습니다.

```java
try{
    //...   
}
catch (ExceptionA | ExceptionB e) {
    e.printStackTrace();
}
```

만약 예외 클래스가 조상과 자손의 관계에 있다면 컴파일 에러가 발생합니다.
```java
try{
    //...   
}
catch (ParentException e | ChildException e) {
    e.printStackTrace();
}
```

## 5. 예외 발생시키기
키워드 ```throw``` 를 사용해서 프로그래머가 고의로 예외를 발생시킬 수 있습니다.

1. 우선, 연산자 new 를 이용해서 발생시키려는 예외 클래스의 객체를 만든다.
    ```java
    Exception e = new Exception("고의로 발생시킴.");
    ```
   
2. 키워드 throw 를 이용해서 예외를 발생시킨다.
    ```java
    throw e;
    ```
   
1, 2번을 ```throw new Exception("고의로 발생시킴.")"``` 처럼 한 줄로 처리가 가능합니다.

## 6. 메서드에 예외 선언하기
try-catch 문을 사용하는 예외처리 외에, 예외를 메서드에 선언하는 방법이 있습니다.
메서드의 선언부에 키워드 throws 를 사용해서 메서드 내에서 발생할 수 있는 예외를 적어주기만 하면 됩니다. 그리고 예외가 여러 개일 경우에는 쉼표(,) 로 구분합니다.

```java
void method() throws Exception1, Exception2, Exception3, ... ExceptionN {
    // 내용
} 
```

메서드의 선언부에 예외를 선언함으로써 메서드를 사용하려는 사람이 메서드의 선언부를 보았을 때, 이 메서드를 사용하기 위해서는
어떠한 예외들이 처리되어져야 하는지 쉽게 알 수 있습니다.

예외가 발생한 메서드에서 예외처리를 하지 않고 자신을 호출한 메서드에게 예외를 넘겨줄 수는 있지만, **try-catch 문을 통해 처리하지 않는다면 비정상적인 종료가 발생**합니다.

