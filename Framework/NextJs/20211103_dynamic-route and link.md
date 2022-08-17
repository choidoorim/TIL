# Dynamic Router, next/link
오픈 API 를 활용하기 위해서 axios 를 설치한다.

```npm i axios```

## Dynamic Router
```javascript
// pages/example/[id].js
import { useRouter } from "next/router";

export default function About() {
    const router = useRouter();
    const {id} = router.query;
    return (
        <div>
            <p>Post: {id}</p>
        </div>
    )
}
```

위의 코드처럼 작성하고 ```localhost:3000/example/30``` 으로 접속 시 30 이 출력 되는 것을 확인할 수 있다.

![1](https://user-images.githubusercontent.com/63203480/139853521-43519260-b01b-44f9-9d15-751cec039d6c.PNG)

## next/link
Link 태그와 동일한 기능으로 원하는 링크로 이동할 수 있도록 한다.

```javascript
// ItemList.js
// ...
<Link href={`/example/${item.id}`}>
// ...
```

```javascript
// example/[id].js
import { useRouter } from "next/router";

export default function About() {
    const router = useRouter();
    const {id} = router.query;

    return (
        <div>
            <p>Post: {id}</p>
        </div>
    )
}
```

위의 2개의 파일 처럼 작성할 경우, 해당 Link 에 해당하는 것이 실행될 경우
2번째 코드의 파일 링크로 이동한다.
