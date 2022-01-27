# async/await Promise All
## Promise All 
순서와 무관한 여러 개의 비동기 처리를 병렬로 처리하기 위해 사용하는 기능입니다.
예를 들면, 하나의 상품 키 값을 가지고 **"상품을 조회하고, 그 상품에 대한 리뷰들을 조회"** 하는 경우입니다.

덧셈, 뺄셈, 나눗셈 3 개의 비동기 처리를 하는 예제로 진행했습니다.
## Example
### 개별로 처리
```javascript
const calcPlus = async (a, b) => {
    return a + b;
}

const calcMinus = async (a, b) => {
    return a - b;
}

const calcDivide = async (a, b) => {
    return a / b;
}

const promiseAllTest = async () => {
    const a = 8;
    const b = 2

    const promise1 = await calcPlus(a, b);
    const promise2 = await calcMinus(a, b);
    const promise3 = await calcDivide(a, b);
    
    console.log(promise1, promise2, promise3); // 10 6 4
}

promiseAllTest();
```

위의 방식은 각각의 함수가 순차적으로 실행됩니다. 

### Promise.all 
```javascript
const calcPlus = async (a, b) => {
    return a + b;
}

const calcMinus = async (a, b) => {
    return a - b;
}

const calcDivide = async (a, b) => {
    return a / b;
}

const promiseAllTest = async () => {
    const a = 8;
    const b = 2
    const calculations = [
        await calcPlus(a, b),
        await calcMinus(a, b),
        await calcDivide(a, b),
    ]

    const result = await Promise.all(calculations);
    console.log(result); // [10, 6, 4]
}

promiseAllTest();
```

순차적으로 함수를 실행시킨 것과 차이점은 함수들이 앞의 함수의 실행이 완료되는 것을 기다리지 않고 병렬적으로 처리한다는 것입니다.

![all-fullfilled-9](https://user-images.githubusercontent.com/63203480/151383489-b68606fe-1d5f-4266-ba9a-2b3740b2c654.svg)     
출처: https://dmitripavlutin.com/promise-all/

위의 사진은 Promise All 에 대해서 잘 표현한 그림입니다.
즉, Promise All 을 사용한다면 여러 함수들을 병렬로 처리해서 빠르게 처리할 수 있다는 것입니다.

빠르다는 분명한 장점이 있지만 단점도 있습니다.

## reject
병렬로 처리되는 함수들 중에서 ERROR 가 발생할 경우, ERROR 케이스로 빠지게 됩니다. 
만약 4 개의 함수중에 ERROR 가 발생한 1 개의 케이스가 있을 경우 Promise All 이 나머지 값들을 성공 여부와 상관 없이 즉시 ERROR 로 처리한다는 뜻입니다.
그렇게 되면 에러가 발생하면 다른 함수들의 결과는 무시됩니다.    
즉, ```Promise.all()``` 은 모 아님 도 일 때 사용해야 된다는 것입니다.

이러한 Promise All 의 문제점을 해결한 기능이 있습니다.

## Promise.allSettled
추가된지 얼마 되지 않은 기능으로 Promise All 과 달리 모든 함수가 처리될 때까지 기다린 후,
그에 대한 결과를 반환 한다고 합니다.

Javascript 에서는 비동기 처리에 대한 다양한 기능들을 제공해주므로 상황에 따라서 적절한 기능을 

참고 자료
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
https://code-masterjung.tistory.com/91
https://ko.javascript.info/promise-api
