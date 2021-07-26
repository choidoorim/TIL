### :clipboard: Promise
- Javascript 에서 비동기를 간편하게 처리할 수 있도록 도와주는 Object 이다.
  

- 정해진 시간에 기능을 수행하고 나서 정상적으로 기능이 실행됐다면, 성공적인 메시지와 함께 결과 값을 전달해준다.
만약 중간에 예상치 못한 문제가 발생할 경우 Error 메시지를 전달해줍니다
  

- Promise 는 Class 이기 때문에 new 키워드를 통해 생성해야 한다.

```javascript
const promise = new Promise((resolve, reject) => {
          // doing some heavy work
          console.log('doing something...')
        }
);
```
- Promise 를 만드는 순간  executor 라는 Callback 함수가 자동으로 실행되어 사용자가 요구하지 않은 요청도 처리될 수 있기 때문에 
  유의하여 사용해야 한다 (중요)
  

- network 에서 데이터를 가져오거나, 파일을 읽는 등의 시간이 오래 걸리는 연산들을 동기적으로 처리하게 된다면 연산이 수행되는 동안 다음라인의 
코드가 실행되지 않기 때문에 시간이 걸리는 연산들은 Promise 를 사용해서 비동기적으로 처리하는 것이 좋다. 


- Promise 의 중요한 2가지 point
  1. state : 프로세스가 무거운 연산을 실행하고 있는지, 기능 수행이 끝나서 완료됐는지 등 이런상태에 대해서 이해하는 것이 중요하다
  2. producer(데이터 제공) 와 consumer(제공 된 데이터를 사용) 의 차이점을 알고 있는 것이 중요하다.
  
#### State
- Pending State : 연산이 수행중인 상태
  

- Fulfilled State : 연산이 성공적으로 수행 된 상태
  

- Rejected State : 파일을 찾을 수 없거나, network 문제 등 Error 가 발생한 상태


#### Producer
- 우리가 원하는 기능을 수행해서 해당하는 데이터를 만들어내는 것
```javascript
const promise = new Promise((resolve, reject) => {
  // doing some heavy work
  console.log('doing something...')
  setTimeout(() => {
    // resolve('Alvin');                    // then 결과를 보고 싶다면 주석 해제
    // reject(new Error('no network'));     // catch 결과를 보고 싶다믄 주석 해제
  }, 2000)
});
```

- 성공적으로 수행된 연산의 결과 혹은 데이터를 ```resolve``` 라는 callback 함수를 통해 전달하면 된다
  

- 에러가 발생 될 경우 ```reject``` callback 함수에 Error 라는 Object 를 통해 값을 전달한다
  - Error Object : Javascript 에서 제공하는 Object


- Promise 사용 시 resolve, reject 처리를 잘 해줘야 Pending 상태에 걸리지 않게 된다

#### Consumer
- 만들어낸 데이터를 소비하는 것
```javascript
promise
  .then((value) => {
    console.log(value)
  })
  .catch((error) => {
    console.log(error)
  })
  .finally(() => {
    console.log('finally ')
  });
```
- then 함수 : 정상적으로 처리 됐을 경우 resolve callback 함수에서 전달 된 값이 value 에 들어오게 된다
- catch 함수 : 에러가 발생할 경우 reject 를 통해 error 값을 받아와서 처리한다
- finally 함수 : 성공, 실패와 상관 없이 어떤 기능을 마지막에 수행하고 싶을 때 사용한다

#### Promise chaining
- 아래의 예제 코드와 같이 다른 비동기적 코드를 묶어서 처리도 가능하다
```javascript
const fetchNumber = new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000);
});

fetchNumber
.then(value => value * 2)   // 1 * 2 = 2
.then(value => value * 3)   // 2 * 3 = 6
.then(value => {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve(value - 1), 1000); // 6 - 1 = 5
    })
})
.then(value => console.log(value)) // result : 5, total time = 1000 + 1000 ms
```

#### Error Handling
- catch 를 이용해서 Error 가 발생 시 유연하게 처리해야 한다
```javascript
const getHen = () => new Promise((resolve, reject) => {
    setTimeout( () => resolve('Chicken'), 1000);
});
const getEgg = hen => new Promise((resolve, reject) => {
    setTimeout( () => reject(new Error(`error ${hen} => Egg`)), 1000);
});
const cook = egg => new Promise((resolve, reject) => {
    setTimeout(() => resolve(`${egg} => Fried egg`), 1000)
})
getHen()
    .then(getEgg)
    .catch(error => {
        return 'Bread';     // Error 가 발생하면 Bread 를 Return 한다, 바로바로 catch 를 작성해서 Error 를 잘 Control 해야 한다
    })
    .then(cook)
    .then(console.log)
    .catch(console.log)
```

- 아래 두 코드는 같은 코드이다
  ```javascript
  getHen()
        .then(hen => getEgg(hen))   
        .then(egg => cook(egg)) 
        .then(meal => console.log(meal));
  ```
  ```javascript
  getHen()
        .then(getEgg)   
        .then(cook) 
        .then(console.log);
  ```
  
#### Callback Chain => Promise 변경
```javascript
class UserStorage {
    loginUser (id, password) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (
                    (id === 'Alvin' && password === 'alvin') ||
                    (id === 'coder' && password === 'coding')
                ) {
                    resolve(id);
                } else {
                    reject(new Error('not found'));
                }
            }, 2000)
        })
    }

    getRoles (user) {
        return new Promise((resolve, reject) => {
            setTimeout( () => {
                if (user === 'Alvin') {
                    resolve({name: 'Alvin', role: 'admin' });
                } else {
                    reject(new Error('no Access'));
                }
            }, 1000);
        })
    }
}

const userStorage = new UserStorage();
const id = 'Alvin';
const pw = 'alvin';
userStorage.loginUser(id, pw)
.then(userId => userStorage.getRoles(userId)) // userId 가 전달 됌
.then(user => console.log(`Name is ${user.name}, Role is ${user.role}`))    //
.catch(error => console.log(error))
```
