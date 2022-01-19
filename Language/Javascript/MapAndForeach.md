# map 과 forEach 의 차이점
```forEach``` 와 ```map``` 은 배열의 요소들을 함수에 넣고 실행하는 과정은 같지만,
```map``` 은 그 결과에 return 값이 있지만, ```forEach``` 는 return 을 보내지 않습니다.

for 문을 통해서도 충분히 해결 가능하지만 이러한 map 이나 forEach 와 같은 선언적 프로그래밍 스타일은 
표현력이 뛰어나고 작성하기 쉽고 훨씬 읽기 쉽습니다. 
많은 이유로 좋지만 성능이 중요한 경우에는 그렇지 못합니다. for 문은 일반적으로 선언적 루프보다 3배 이상 빠르다고 합니다.    
대부분의 응용 프로그램에서 큰 차이를 보이지는 않지만, 비디오 처리, 과학 계산 또는 게임 엔진에서 대량의 데이터 처리에서는 
전체 성능에 큰 영향을 미칩니다.    
위의 경우가 아니면 대부분은 단순한 선언적 프로그래밍이 좋습니다. 상황에 따라 최상의 선택을 하는 것이 좋을 것 같습니다.

밑에 사진은 벤치마크 라이브러리를 사용해서 테스트한 결과라고 합니다. 아래 초당 작업수가 높을수록 좋은 것입니다.

## foreach vs for/for..of

![forEach](https://user-images.githubusercontent.com/63203480/150138553-107f8157-d0fe-48f4-bd38-c8a14a795dbd.png)

## map vs for vs for..of

![map](https://user-images.githubusercontent.com/63203480/150138852-4d2f8b44-5719-45f3-a92e-b9f33ebe8a09.png)

## forEach
```forEach()``` 메서드는 아무것도 return 하지 않습니다. 그리고 배열 요소를 각각 개별적으로 실행합니다.

```javascript
const array = [
    {
        id: 1,
        name: 'a',
    },
    {
        id: 2,
        name: 'b',
    },
    {
        id: 3,
        name: 'c',
    }
];

array.forEach((arr) => console.log(arr));
```

```
{ id: 1, name: 'a' }
{ id: 2, name: 'b' }
{ id: 3, name: 'c' }
```

## map
배열 내의 모든 각각의 값들에 대해서 오름차순으로 접근하여 새로운 값을 정의하고 새로운 배열을 만들어서 반환합니다.

### Ex1
```javascript
const array = [1, 2, 3, 4];

const new_array = array.map((num) => num * 2);
console.log(new_array);
```

```
[ 2, 4, 6, 8 ]
```

### Ex2
```javascript
const array = [
    {
        id: 1,
        name: 'a',
    },
    {
        id: 2,
        name: 'b',
    },
    {
        id: 3,
        name: 'c',
    }
];

const new_array = array1.map((obj) => {
    return { 'testId': obj.name };
});
```

```
[ { testId: 'a' }, { testId: 'b' }, { testId: 'c' } ]
```

참고: https://leanylabs.com/blog/js-forEach-map-reduce-vs-for-for_of/