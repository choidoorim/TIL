# SSR and Dynamic Rendering
NextJS 는 모든 페이지를 사전에 렌더링(Pre-rendering) 한다.
따라서 더 좋은 퍼포먼스와 검색엔지최적화(SEO) 가 가능하다.

Pre-Rendering 에는 

1. 정적 생성
2. SSR(Server Side Rendering, Dynamic Rendering)     

2가지가 있다. 2가지의 차이점은 언제 html 파일을 생성하는가이다.

## 정적생성
- 프로젝트가 빌드하는 시점에 html 파일들이 생성된다.
- 모든 요청에 재사용을 한다. 즉, 파일을 만들어 놓고 재사용한다는 뜻이다.
- 성능 때문에 Next JS 는 정적 생성을 권고한다.
- 정적 생성된 페이지들을 CDN 에 캐시된다.
- ```getStaticProps``` / ```getStaticPaths```
- 블로그 게시글, 도움말 문서, 마케팅 페이지 등

## SSR
- 매 요청마다 html 을 생성한다.
- 항상 최신 상태를 유치할 수 있다.
- ```getServerSideProps```
- 제품 상세페이지 등 자주 바뀌는 경우 사용.

유저의 요청전에 페이지를 생성해도 된다면 **정적생성** 을 사용한다.
반면에 항상 최신 상태의 데이터를 유지하기 때문에 조금 느리더라도 그런점에서는 **SSR** 을 사용한다.

```javascript
const Post = ({item}) => {
    return  <>{item && <Item item={item}/>}</>;
};
export async function getServerSideProps(context) {
    const id = context.params.id;
    const apiUrl = `http://makeup-api.herokuapp.com/api/v1/products/${id}.json`;
    const res = await Axios.get(apiUrl);
    const data = res.data;

    return {
        props: {
            item: data,
        },
    };
}
```

위의 코드처럼 context 를 통해 데이터를 미리 받아서 처리 후, Post 메서드에게 넘길 수 있다.

## next/router
```next/router``` 모듈의 router.push 메서드를 통해 URL 을 이동시켜줄 수 있다.

```javascript
import {useRouter} from "next/router";
/...
function goLink(e, data) {
    if (data.name === "home") {
        router.push("/");   // 해당 링크로 이동
    } else if (data.name === "about") {
        router.push("/about");
        //location.href = "/about";
    }
}
/...
```

```location.href``` 태그를 사용하지 않는 이유는 새로고침되기 때문에 SPA 의 빠른 장점이 사라져 ``next/router`` 나 ```next/link``` 를 사용한다.

## Hook: useEffect
유저가 해당 페이지에 접속시 ```useEffect()``` 에 넣은 function 을 실행하도록 한다.

```javascript
import { useEffect, useState } from "react";
/...
function checkLogin() {
    Axios.get("/api/isLogin").then((res) => {
        if (res.status === 200 && res.data.name) {
            //로그인
        } else {
            //로그인 안됨
        }
    });
}

useEffect(() => {
    checkLogin();
}, []);
/...
```