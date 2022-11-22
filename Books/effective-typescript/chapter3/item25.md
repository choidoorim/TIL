# 아이템 25. 비동기 코드에는 콜백 대신 async 함수 사용하기

과거에 자바스크립트에서는 비동기 동작을 모델링하기 위해서 콜백을 사용했다. 그렇기에 콜백지옥이라는 것을 필연적으로 마주하게 되었다.

```tsx
fetchURL(ur11 , function(responsel) { 
	fetchURL(ur12, function(response2) {
		fetchURL(ur13, function(response3) {
			//...
		}
	}
}
```

ES2015 에서는 콜백지옥을 극복하기 위해 Promise 라는 개념을 도입했다.

```tsx
const page1Promise = fetch(url);
page1Promise.then(reponse1 => {
	return fetch(url2);
}).then(response2 => {
	return fetch(url3);
}
```

Promise 를 통해 실행 순서와 코드의 순서가 동일할 수 있게 되었다. 또한 오류를 처리하기도 하고 `Promise.all` 과 같은 기법도 사용이 가능해졌다.

ES2017 에서는 async await 키워드를 도입해 콜백지옥을 더욱 더 간단하게 처리할 수 있게 되었다.

```tsx
async function fetchPages() {
	const responsel = await fetch(urll); 
	const response2 = await fetch(url2); 
	const response3 = await fetch(url3);
	//...
}
```

`await` 키워드는 각각의 `resolve` 가 해결될 때까지 `fetchPages` 함수의 실행을 멈춘다. 만약 `reject` 되면 예외를 던진다. 이를 통해서 일반적인 `try-catch` 문을 사용할 수 있다.

ES5 또는 더 이전 버전을 대상을 할 때, 타입스크립트 컴파일러는 `async/await` 가 동작되도록 정교한 변환을 수행한다. 즉, **타입스크립트는 런타임에 관계없이 `async/await` 를 사용할 수 있다는 것**이다.

콜백보다는 `Promise` 나 `async/await` 를 사용해야 하는 이유는 **코드를 작성하기 용이**하고 **타입을 추론**하기 쉽기 때문이다.

`[Promise.race](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)` 도 타입추론과 잘 맞는다. `Promise.race` 에 timeout 을 추가하는 패턴을 흔하게 사용하는 패턴이다.

```tsx
function timeout(millis: number): Promise<never> {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject('timeout'), millis)
  })
};

async function fetchWithTimeout(url: string, ms: number) {
  return Promise.race([fetch(url), timeout(ms)]);
}
```

fetch 와 timeout 중에 먼저 완료된 것을 처리하도록 하는 코드이다. 타입구문이 없어도 `fetchWithTimeout` 의 Return Type 은 `Promise<Response>` 로 추론된다.

`Promise.race` 의 Return Type 은 `Promise.race` 배열안에 입력한 값들에 대한 유니온 타입이다. `Promise<Response | never>` 가 되지만 **never 타입은 공집합으로 유니온과 아무런 관계가 없기 때문**에 `Promise<Response>` 가 된다.

간혹 위와 같이 Promise 객체를 직접생성해서 사용해야 하는 경우가 있다. 하지만 일반적으로는 async/await 를 사용하는 것이 좋다. 간결하고 직관적인 코드가 되며, `**async` 함수는 항상 `Promise` 를 반환하도록 강제**하기 때문이다.

async 함수에서 Promise 를 반환한다면 또 다른 Promise 로 래핑되지 않는다. `Promise<Promise<T>>` 가 아닌 `Promise<T>` 가 되는 것이다. 어떤 함수가 Promise 를 반환한다면 async 로 선언하는 것이 좋다.
