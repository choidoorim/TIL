### :clipboard: function

- function 은 기본적으로 input 을 받아서 잘 처리한 후 output(return) 을 한다
- 함수는 function Name, input, output 을 잘 작성하는게 중요하다

#### function declaration

- function name (params1, params2 ...) { body ... return; }
- 한 가지의 함수는 한 가지의 일만 수행하도록 하여야 한다
- 함수 명은 어떤 것을 수행하는 역할을 하므로, 동사형태 혹은 command 형식으로 지정하는 것이 좋다 :smile:
- Javascript 에서 function 은 Object 로 간주된다
  - 따라서 변수에 할당이 가능하고 파라미터로 전달이 가능하며 함수를 return 할 수도 있는 것이다

#### Parameters
- 일반 값 (premitive type) 같은 경우는 value 가 전달이 된다
- object 같은 경우에는 reference 가 전달이 된다
```javascript
function ChangeName(obj, name) {
    obj.name = name;
}

const alvin = { name : 'alvin' };
ChangeName(alvin, 'choi') // ref, value
console.log(alvin)
```

#### Default Parameters
- function Name (params1 = 'default value')
  - 위와 같이 function parameter 의 default 값을 지정해 줄수 있다
  
```javascript
function showMessage(message, from = 'unknown') {   // function parameter 에 default 값을 지정할 수 있다
    console.log(`${message} by ${from}`)
}
showMessage('Hi!') // Hi! by unknown 출력 됌
```

#### Rest Parameters
- function Name (...args) : '...'을 사용
```javascript
function printAll (...args) {
    for (let i = 0; i < args.length; i++) {
        console.log(args[i])
    }
}
printAll('alvin', 'Hello', 'coding')
```

#### Local Scope
- 전역변수(= 밖), 지역 변수(=안) : 밖에서는 안이 보이지 않고 안에서만 밖을 볼 수 있다
- Javascript 에서 scope 는 유리창에 틴트처리가 되어 안에서만 밖을 볼 수 있는 것과 같다

#### Return a value
- return type 이 없는 함수들은 return undefined 이 들어가 있는 것과 같다
```javascript
function sum(a, b) {
    return a + b
}

console.log(`sum: ${sum(1 , 2)}`)
```

#### function Expression
- 함수를 선언 하는 것은 hoisting 이 발생한다
  - hoisting: Javascript 엔진이 선언 된 것을 맨 위로 올리기 때문에 로직의 순서에 상관 없이 선언되기 전에 실행이 가능한 것
```javascript
sum(1, 2)
function sum(a, b) {
return a + b
}
```

- 하지만 함수를 변수에 정의 할 경우에는 할당 된 후 부터 함수의 사용이 가능하다.
```javascript
// print(); - > (X)
const print = function () {
    console.log('print')
}
print();
```

#### callback function
- parameter 와 상황에 따라 전달 된 함수를 호출하는 것이 callback function 이다
- 아래 예제에서는 printYes 와 printNo 가 callback function 이다
- 아래 printNo expression 에서 함수에 이름을 쓰는 것은 디버깅 시 함수 명이 나오도록 하기 위해서이다

```javascript
function randomQuiz(answer, printYes, printNo) {
    if (answer === 'i love coding') {
        printYes();
    }
    else {
        printNo();
    }
}
const printYes = function () {
    console.log('yes!')
}

const printNo = function print () {
    console.log('no!')
}

randomQuiz('i love coding', printYes, printNo)
```

#### Arrow function
- const arrow = (params1, params2 ...) => return 할 logic
- 항상 이름이 없는 anonymous function 이다
- 코드를 간결하게 만들어 준다
```javascript
const expressionPrint = function () {
    console.log('expressionPrint')
}

const arrowPrint = () => console.log('arrowPrint');
const add_num = (x, y) => x + y; 
expressionPrint();
arrowPrint();
console.log(add_num(1, 2));
```