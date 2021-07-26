### :clipboard: Array and APIs
#### 자료구조
- 비슷한 데이터들을 한 군데에 모아서 보관하는 것
- Javascript 는 동적인 언어이기 때문에 다양한 type 들이 자료구조 안에 담길 수 있다
  따라서 같은 type 이 배열에 담길 수 있도록 설계해야 한다
  - 원래는 동일한 type 의 Object 만 담을 수 있다
  
#### Array Declaration
- 배열 선언하는 2가지 방법
```javascript
const arr1 = new Array();
const arr2 = [1, 2]
```

#### Index Position
- 배열의 마지막 값 접근 방법 : ```array[array.length - 1]```
```javascript
const fruits = ['apple', 'banana'];
console.log(fruits);
console.log(fruits.length);
console.log(fruits[0]);
console.log(fruits[fruits.length - 1]);
```

#### Looping
- 배열에 for 문을 사용하는 방법
1. for
```javascript
const fruits = ['apple', 'banana'];
for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i])
}
```

2. for of
```javascript
const fruits = ['apple', 'banana'];
for (let fruit of fruits) {
    console.log(fruit)
}
```

3. for Each - callback 함수를 사용한다
* 함수를 호출하거나 사용할 때는 직접 선언된 곳으로 가서 어떤 함수인지, 어떤 파라미터를 사용하는지
어떤 것을 return 하는지 등을 보면서 코딩하는 습관을 기르자!
* 'thisArg? any' 처럼 물음표가 써져 있는 것은 사용해도 되고 사용하지 않아도 되는 것이다
```javascript
fruits.forEach(function (fruit, index, array){
    console.log(fruit, index, array)
})
```

#### Addition, deletion, copy
- unshift 와 shift 는 push, pop 보다 느리다!!
  왜냐하면 앞에 데이터를 넣을 경우 기존 값들을 옮기는 작업이 필요하기 때문이다.
  

- Addition
1. push: item 의 맨 뒤에 값 추가
```javascript
fruits.push('melon')
```
2. unshift: item 의 맨 앞에 값 추가
```javascript
fruits.unshift('peaches')
```

- Deletion
1. pop: item 의 맨 뒤에 값 삭제
```javascript
fruits.pop()
```
2. shift: item 의 맨 앞에 값 삭제
```javascript
fruits.shift()
```
3. splice: 지정 된 위치의 item 값 삭제
- 몇 개를 삭제 할 것인지 지정해주지 않으면 해당 index 값 제외하고 모든 수를 삭제한다
```javascript
fruits.splice(1) // 1 위치의 index 제외하고 모든 수 삭제
fruits.splice(0, 2) // 0 위치의 index부터 2개의 값을 삭제
fruits.splice(0, 2, 'strawberry') // 지운 위치에 'strawberry'를 추가
```
```javascript
const num = [1, 2, 3 ,4, 5]
const result = num.splice(0, 2)
console.log(result) // 1, 2
console.log(num)    // 3, 4, 5
```

#### Combine two arrays
- ```array1.concat(array2)``` : array1 에 array2 를 합친다
```javascript
const fruits2 = ['apple', 'melon']
const newFruits = fruits.concat(fruits2)
```

#### Searching
- ```array.indexOf(element)``` : element 가 어느 위치 index 에 있는지 알려준다. 존재하지 않을 경우 -1 을 return 한다.
만약 동일한 element 가 있을 경우 앞에 있는 index 가 return 된다
- ```array.lastIndexOf(element)``` : 제일 마지막 element 가 어느 위치 index 에 있는지 알려준다.
```javascript
const fruits = ['apple', 'melon', 'apple']
fruits.indexOf('apple') // 0
```
```javascript
const fruits = ['apple', 'melon', 'apple']
fruits.lastIndexOf('apple') // 2
```

- ```array.includes(element)``` : element 가 존재하는지 알려준다(True or False)
```javascript
const fruits = ['apple', 'melon']
fruits.includes('apple') // True
fruits.includes('banana') // False
```

#### Sort
- `array.sort()` : 배열을 정렬해준다. 하지만 아스키코드 값으로 정렬하기 때문에 제대로 정렬이 안되기도 한다
```javascript
const array = [1, 111, 2323, 4444, 23, 11]
array.sort() // [ 1, 11, 111, 23, 2323, 4444 ]
```

- 오름차순 정렬
```javascript
const array = [1, 111, 2323, 4444, 23, 11]
array.sort(function(a, b) {
    return a - b
})
array.sort((a, b) => a - b) // [ 1, 11, 23, 111, 2323, 4444 ]
```
- 내림차순 정렬
```javascript
const array = [1, 111, 2323, 4444, 23, 11]
array.sort(function(a, b) {
    return b - a
})
array.sort((a, b) => b - a) // [ 4444, 2323, 111, 23, 11, 1 ]
```

#### join Method
- ```join('구분자')``` : 배열에 있는 요소들을 하나의 string 으로 return 해준다.
구분자를 지정하지 않을 경우 ',' 가 구분자가 된다
```javascript
const fruits = ['apple', 'banana', 'orange']
const result = fruits.join('/')
console.log(result) // apple/banana/orange
```

#### split Method
- ```split('구분자')``` : 구분자를 기준으로 주어진 string 을 배열로 변환해준다
```javascript
const fruits = 'apple, banana, orange'
const result = fruits.split(',',2)
console.log(result) // [ 'apple', ' banana' ]
```

#### reverse Method
- ```reverse()``` : 배열 안에 있는 item 의 순서를 거꾸로 만든다
```javascript
const num = [1, 2, 3, 4, 5]
console.log(num.reverse())
```

#### slice Method
- ```slice(start index, end index)``` : 배열에서 원하는 부분만 return 할 때 사용하는 것으로 start index 부터 (end index - 1) 까지 return 해준다
```javascript
const num = [1, 2, 3 ,4, 5]
    const result = num.slice(2, 5)
    console.log(result)
```

#### find Method
- ```find()``` : 배열 내에서 첫번 째로 찾아진 요소를 return 한다. 없을 경우 undefined 를 return 한다
```javascript
class Student {
    constructor(name, age, enrolled, score) {
        this.name = name;
        this.age = age;
        this.enrolled = enrolled;
        this.score = score;
    }
}
const students = [
    new Student('A', 29, true, 45),
    new Student('B', 28, false, 80),
    new Student('C', 30, true, 90),
    new Student('D', 40, false, 66),
    new Student('E', 18, true, 88)
];
{
    const result = students.find((student, index) => student.score === 90)
    console.log(result)
}
```

#### filter Method
- ```filter()``` : 배열 내에서 우리가 원하는 값만 받아올 수 있다
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
{
    const result = students.filter((student) => student.enrolled
    );
    console.log(result);
}
```

#### map Method
- ```map()``` : 배열에 있는 요소들을 콜백 함수에서 처리 된 새로운 값으로 변환한다
- callback 함수들로 지정된 인자는 알아보기 쉽도록 선언하는 것이 중요하다
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
{
    const result = students.map((student) => student.name)
    console.log(result) // ['A','B','C','D','E']
}
```

#### some Method
- ```some()``` : 배열에 있는 요소 중 콜백함수의 조건을 충족하는 요소가 있을 경우 true 를 return 한다
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
const result = students.some(function (student) {
    return student.score < 50
});
console.log(result)
```

#### every Method
- ```every()``` : 배열에 있는 모든 요소가 콜백함수의 조건에 만족하면 true 를 return 한다
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
const result2 = students.every(function (student) {
    return student.score < 50
})
console.log(result2)
```

- 배열에서 하나라도 만족하는 것이 있는지 확인 할 때는 some 을 이용하면 되고, 모두 만족하는 것이 있는지 확인할 때는 every 를 이용하면 된다

#### reduce Method
- ```reduce()``` : 시작점 부터 어떤 배열에 있는 값들을 누적 시킬 때 사용한다
- ```reduceRight()``` : 배열에 끝 부터 시작까지 값들을 누적 시킨다
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
const result = students.reduce((prev, curr) => {
    return prev + curr.score    // 이전 값  + 현재 값
}, 0)
console.log(result) // 369
```

#### 함수형 프로그래밍 예제
```javascript
class Student {
  constructor(name, age, enrolled, score) {
    this.name = name;
    this.age = age;
    this.enrolled = enrolled;
    this.score = score;
  }
}
const students = [
  new Student('A', 29, true, 45),
  new Student('B', 28, false, 80),
  new Student('C', 30, true, 90),
  new Student('D', 40, false, 66),
  new Student('E', 18, true, 88)
];
{
    const result = students
        .map(student => student.score)      // 점수로 변환
        .sort( (a, b) => a - b)             // 오름차순으로 정렬
        .join()                             // string 으로 변환
    console.log(result)                     // 45, 66, 80, 88, 90
}
```