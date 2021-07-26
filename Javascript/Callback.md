### :clipboard: Callback function
#### 동기와 비동기의 차이점
- Javascript 는 동기적이다 (= 동기적: hoisting 다음에 우리가 짠 코드의 순서에 맞춰서 실행된다)
  - hoisting : var, function declaration 이 코드 상 맨위로 올라가는 것 
  
- 비동기적(asynchronous) : 코드의 순서에 상관 없이 언제 코드가 실행될 지 모른다
  

- Sync Example
```javascript
console.log(1)  // sync
console.log(2)  // sync
console.log(3)  // sync
```
- Async Example
```javascript
console.log(1)  // sync
setTimeout(() => console.log(2), 1000);// async
console.log(3)  // sync
// result : 1 3 2
// 1000ms(1초) 가 지난 후에 setTimeout 메소드에 전달한 함수를 다시 불러달라는 의미(callback)
```
- Callback 은 Async 일 때만 사용하는 것이 아니다. 
  1. Sync Callback
  2. Async Callback
  ```javascript
  console.log(1)  // sync
  setTimeout(() => console.log(2), 1000);   // async
  console.log(3)  // sync
  // Sync Callback
  function printImmediately(print) {
  print();
  }
  printImmediately(() => console.log('Sync callback'))
  
  // Async Callback
  function printWithDelay(print, timeout) {
  setTimeout(print, timeout)
  }
  printWithDelay(() => console.log('Async callback'), 2000)
  // result 1, 3, Sync callback, 2, Async callback
  ```
  
- Javascript 는 함수를 callback 형태의 인자로 다른 함수에 전달할 수 있고, 다른 변수에 할당 할 수도 있다

#### Callback Chain
```javascript
class UserStorage {
    loginUser (id, password, onSuccess, onError) {
        setTimeout(() => {
            if (
                (id === 'Alvin' && password === 'alvin') ||
                (id === 'coder' && password === 'coding')
            ) {
                onSuccess(id);
            } else {
                onError(new Error('not found'));
            }
        }, 2000);
    }

    getRoles (user, onSuccess, onError) {
        setTimeout( () => {
            if (user === 'Alvin') {
                onSuccess({name: 'Alvin', role: 'admin' });
            } else {
                onError(new Error('no Access'));
            }
        }, 1000);
    }
}

const userStorage = new UserStorage();
const id = 'alvin'
const pw = 'alvin'
userStorage.loginUser(
    id,
    pw,
    user => {
        userStorage.getRoles(
            user,
            userWithRole => {
                console.log(`Hello ${userWithRole.name}, you have a ${userWithRole.role} role`);
            },
            error => {
                console.log(error)
            }
        );
    },
    error => {
        console.log(error)
    }
);
```
- 콜백 지옥의 문제점
  1. 가독성이 너무 떨어진다 : 어디서 어떻게 연결되어 있는지 한 눈에 보기 어렵다
  2. Business Logic 을 한 눈에 알아보기도 너무 어렵다
  3. Error 발생 시, 혹은 디버깅 시에 굉장히 어려워진다. Chain 이 길어질 수록 문제점을 분석하는게 힘들다
  4. 유지보수가 어렵다