### :clipboard: let 과 const
- let: 메모리에서 read, write가 가능하다(mutable)
- const: 메모리에서 read 만 가능하다. 즉, 변수에 값이 할당되면 변경 할 수 없다(immutable)
- <mark style='background-color: #f6f8fa'> 변수를 변경할 이유가 없다면 왠만하면 const 를 사용하자! </mark>
- const 의 장점 : 보안성이 좋다, 유지보수 시 사람의 실수를 줄일 수 있다.


### :clipboard: 메모리에 값이 저장되는 방법
- primitive type: 일반 값이 저장되는 방법
- object type: 객체가 저장되는 방법


### :clipboard: operator
#### increment : ' ++ ' 
- 변수 = ++ 값: 값을 증가 후 변수에 할당
- 변수 = 값 ++: 값을 변수에 할당 후 증가
```javascript
let count = 2
const pre_increment = ++count
console.log(`pre_increment: ${pre_increment}, count: ${count}`)
const post_increment = count++
console.log(`PostIncrement: ${post_increment}, count: ${count}`)
```
#### decrement : ' -- '
- 변수 = ++ 값: 값을 감소 후 변수에 할당
- 변수 = 값 ++: 값을 변수에 할당 후 감소
```javascript
let count = 2
const pre_decrement = --count
console.log(`pre_decrement: ${pre_increment}, count: ${count}`)
const post_decrement = count--
console.log(`PostDecrement: ${post_increment}, count: ${count}`)
```

#### Assignment operators
- ' x += y ' 와 ' x = x + y' 는 같은 뜻이다
```javascript
let x = 3
let y = 6
x += y  // x = x + y
x -= y
x *= y
x /= y
```

#### Comparison operators
- '<', '>', '<=', '>='

#### Logical operators
- || (or) : a || b || c -> a, b, c 중 하나만 참이면 된다.
    1. or 연산은 a가 true 일 경우 b, c는 체크하지 않고 멈춘다.
    2. <mark style='background-color: #f6f8fa'> 함수를 체크하는 경우는 제일 마지막에 두는 것이 좋다 :smile: ex) a || b || check() </mark> 
    

- && (and) : a or b or c -> a, b, c 모두 참이여야 한다.
  1. <mark style='background-color: #f6f8fa'> and 연산도 동일하게 함수를 체크하는 경우를 맨 뒤로 두는 것이 좋다 :smile: </mark>
    

- ! (not) : 값을 반대로 바꿔주는 연산자

#### Equality
- ' == ' - loose equality : type 을 신경쓰지 않고 비교한다
    1. 0, '', False 는 loose equality 를 사용하면 동일한 것으로 간주된다
    2. null 과 undefined 는 loose equality 를 사용하면 동일한 것으로 간주된다
- ' === ' - strict equality : type 을 신경 써서 비교한다
    1. 0, '', False 그리고 null 과 undefined 는 type 이 다르기 때문에 strict equality 를 사용하면 다른 것으로 간주된다
- <mark style='background-color: #f6f8fa'> 왠만하면 strict equality 를 사용하는 것이 좋다 :smile: </mark>
```javascript
const stringFive = '5'
const numberFive = 5

console.log(stringFive == numberFive);  // True
console.log(stringFive != numberFive);  // False

console.log(stringFive === numberFive); //True
console.log(stringFive !== numberFive); // False
```

#### Ternary operator
- ' ? ' : condition ? value1 : value2
    1. 조건이 true 일 경우 value1 을 실행하고, False 일 경우 value2가 실행된다
```javascript
const name = 'alvin'
console.log(name === 'alvin' ? 'yes' : 'no'); // yes
```

#### for loop
- for(begin; condition; step)
```javascript
for ( i = 0; i < 10; i++) { // 0 ~ 9
    console.log(`i: ${i}`)
}
```
- for (const arg or args) : args 배열에 있는 값들을 모두 차례대로 arg 변수에 담아준다
```javascript
args = ['alvin', 'choi', 'coding']
for (arg of args) {
    console.log(arg)
}

```