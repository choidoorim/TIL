### :clipboard: Json
- JSON API 가 정의된 곳으로 와서 확인해보면 JSON interface 안에 stringfy, parse 2가지의 method 를 가지고 있는 것을 확인할 수 있다
- 오버로딩: 같은 이름의 method 지만 몇개의 parameter 가 전달되냐에 따라 각각 다른 방식으로 호출이 가능한 것

#### AJAX
- HTTP 를 이용해 서버에서 데이터를 받아올 수 있는 방법
- 브라우저에서 서버와 통신하는 방법은 fetch() API 를 사용하거나 XMLHttpRequest Object 를 사용할 수 있다
- XML 을 사용하게 되면 태그 때문에 파일의 크기도 커지고 가독성이 떨어져 JSON 을 많이 사용한다

#### JSON (JavaScript Object Notation)
- Javascript 의 Object 에서 큰 영감을 받아 만들었기에 key 와 value 로 이루어져 있다
- 브라우저, 모바일, Object 를 file System 에 저장할 때 등에 많이 사용한다

#### 1. stringfy Method
- stringfy() : Object 를 JSON 으로 변환한다


- :bookmark_tabs: Example Code
  - key 와 value 가 전달될 때 처음으로 전달되는 것은 object 를 감싸고 있는 최상위의 것이 전달이 되고, 그 뒤 부터 key 와 value 가 전달된다
  ```
  key : , value : [object Object]
  key : name, value : tori
  key : color, value : white
  key : size, value : null
  key : birthDate, value : 2021-07-19T11:10:47.196Z
  key : jump, value : () => {
  console.log(`${this.name} can jump !`);
  }
  ```
```javascript
let json = JSON.stringify(true);
console.log(json)

json = JSON.stringify(['apple', 'banana']);
console.log(json)                                // ["apple","banana"] : 싱글 포트가 아닌 더블포트로 변환

const rabbit = {
  name : 'tori',
  color : 'white',
  size : null,
  birthDate : new Date(),
  jump : () => {
    console.log(`${name} can jump !`);
  },
};

json = JSON.stringify(rabbit);                    // 함수나 Javascript 에서만 있는 기능들은 변환되지 않는다
console.log(json);

json = JSON.stringify(rabbit, ["name"]);          // 원하는 property 만 json 으로 변환
console.log(json);

json = JSON.stringify(rabbit, (key, value) => {   // 세밀하게 통제하고 싶을 때 callback function 을 사용하면 좋다
  console.log(`key : ${key}, value : ${value}`)
  return value
});                                               // 원하는 property 만 json 으로 변환
console.log(json);
```

#### 2. parse Method
- parse() : JSON 을 Object 로 변환한다


- :bookmark_tabs: Example Code
  - object 를 json 으로 변환하면 method 가 포함되지 않기 때문에 그것을 다시 obj 로 변환하면 당연히 method 는 포함되지 않는다
  ```javascript
  const rabbit = {
    name : 'tori',
    color : 'white',
    size : null,
    birthDate : new Date(),
    jump : () => {
      console.log(`${name} can jump !`);
    },
  };

  json = JSON.stringify(rabbit);
  const obj = JSON.parse(json, (key, value) => {
    console.log(`key : ${key}, value : ${value}`)
    return key === 'birthDate' ? new Date(value) : value
  });
  rabbit.jump();
  obj.jump(); // ERR
  ```
  
```javascript
const rabbit = {
  name : 'tori',
  color : 'white',
  size : null,
  birthDate : new Date(),
  jump : () => {
    console.log(`${name} can jump !`);
  },
};

json = JSON.stringify(rabbit);
const obj = JSON.parse(json, (key, value) => {
    console.log(`key : ${key}, value : ${value}`)
    return key === 'birthDate' ? new Date(value) : value
});
rabbit.jump();
// obj.jump();
console.log(rabbit.birthDate.getDate());
console.log(obj.birthDate);
```