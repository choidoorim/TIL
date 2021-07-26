### :clipboard: Async - Await
- Promise 보다 조금 더 간결하고 간편하게 그리고 동기적으로 실행되는 것처럼 보이게 만들어주는 것이다
- promise 를 깔끔하게 사용할 수 있다
- 무조건 promise 가 나쁘고, async 와 await 로 대체되어야 하는 것은 아니다!

#### Async
- function 앞에 async 를 붙이면 {} 코드블럭이 자동적으로 Promise 로 바뀐다
```javascript
async function fetchUser() {
    return 'Alvin';
}

const user = fetchUser();
console.log(user)
```

#### Await
- await 는 함수 내부에서만 사용이 가능하다.
- await 를 한 코드블럭에서 사용할 때는 병렬화 시켜주는 것이 좋다.
  1. Promise 를 만들어주고
  2. 동기화를 시켜준다
```javascript
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

async function getApple() {
    await delay(1000);
    return 'Apple';
}

async function getBanana() {
    await delay(1000);
    return 'banana'
}

async function pickFruits() {
    try {
        const applePromise = getApple();            // Promise 생성
        const bananaPromise = getBanana();          
        const apple = await applePromise;           // 동기화
        const banana = await bananaPromise;         
        return `${apple} + ${banana}`
    } catch (err) {
        console.log(err);
    }
}

pickFruits().then(console.log)
```

#### useful Promise Method
1. ```Promise.all()``` : 배열로 Promise 들을 전달하게 되면, 모든 Promise 들이 병렬적으로 처리되어 모아주는 기능을 수행한다
```javascript
async function pickFruits() {
    try {
        const applePromise = getApple();            
        const bananaPromise = getBanana();          
        const apple = await applePromise;           
        const banana = await bananaPromise;         
        return `${apple} + ${banana}`
    } catch (err) {
        console.log(err);
    }
}
```
- 위의 동일한 코드와 비교해보면 더욱 더 코드가 깔끔해진 것을 확인할 수 있다
```javascript
function pickAllFruits() {
    return Promise.all([getApple(), getBanana()])
        .then(fruits => fruits.join(' + '))
}
```

2. ```Promise.race()``` : 배열로 전달 된 Promise 들 중에서, 더 빨리 처리 된 Promise 를 return 해주는 기능을 수행한다
```javascript
function pickOnlyOne() {
    return Promise.race([getApple(), getBanana()]);
}

pickOnlyOne().then(console.log)
```