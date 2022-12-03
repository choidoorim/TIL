# 아이템31. 타입 주변에 null 값 배치하기

tsconfig.json 에서 `strictNullChecks` 옵션을 true 로 활성화하면 `null` 이나 `undefined` 관련 오류들이 나타난다. 어떤 변수가 null 이 될 수 있는지 없는지를 타입만으로 명확하게 걸러내기 어렵기 때문에 오류를 걸러내는 if 문을 추가해야된다고 생각할 수 있다.

예를 들어 B 변수가 A 에서 비롯될 때 A 값이 `null` 일 수 없다면 B 도 `null` 일 수 없다. 하지만 반대로 A 가 `null` 일 수 있다면 B 역시 `null` 이 될 수 있다.

```tsx
const A: number = 1;
const B = A; // 반드시 number

const A: number | null = 1;
const B = A; // number or null 가능성이 존재
```

이런 관계는 겉으로 드러나지 않지만 사람과 타입 체커모두 혼란스럽게 한다.

값이 전부 null 이거나 전부 null 이 아닌 경우로 분명하게 구분짓는다면 같이 섞여있을 때 보다 다루기 쉬워진다.

아래는 숫자의 최소, 최대 값을 계산하는 로직이다.

```tsx
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) { // min null check(O), max null check(X)
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

이 코드는 타입 체커에서 통과되지만 오류가 존재한다. 만약 최소, 최대 값이 0 일 경우 값이 덧씌워진다. 예를 들면 `extent([0, 1, 2])` 의 결과는 `[0, 2]` 가 아닌 `[1, 2]` 가 된다.  그리고 nums 가 빈 배열일 경우 `[undefined, undefined]` 를 반환한다. `undefined` 를 포함하는 객체는 다루기 어렵기 때문에 권장하지 않는다.

Typescript 에서 strictNullChecks 옵션을 true 로 활성화 시키면 `extent` 함수의 반환 타입이 `(*number*
| *undefined*)[]` 로 추론되어 설계적 결함이 분명해졌다.

```tsx
else {
  min = Math.min(min, num);
	max = Math.max(max, num); // Argument of type 'number | undefined' is not assignable to parameter of type 'number'. Type 'undefined' is not assignable to type 'number'.ts(2345)
}
```

```tsx
const [min, max] = extent([1, 2, 3]);
const span = max - min; // Object is possibly 'undefined'.ts(2532)
```

`extent` 함수에서 오류가 나는 이유는 `undefined` 을 `min` 에서만 제외하고 `max` 는 하지 않았기 때문이다.

더 나은 해결방법으로는 **min 과 max 를 하나의 객체,배열에 넣고 null 이거나 null 이 아니게 하면 된다**.

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}
```

이렇게되면 extent 함수의 반환 타입이 `[number, number] | null` 이 되기 때문에 처리가 훨씬 수월해진다. 그리고 null 이 아님 단언(non-null assertion) 을 사용하면 `min`, `max` 값을 함수에서 얻을 수 있다.

```tsx
const [min, max] = extent([1, 2, 3])!; // non-null assertion
const span = max - min;
```

하지만 eslint 에서는 **non-null assertion** 는 `strictNullChecks` 의 이점을 이용하지 않기 때문에 권장하지 않는다.

non-null assertion 을 사용하지 않고 조건문(if)을 이용해 타입을 좁혀서 값을 추출하면 된다.

```tsx
const range = extent([1, 2, 3]);
if (range) {
  const [min, max] = range;
  const span = max - min;
}
```

그리고 null 과 null 이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 생긴다.

```tsx
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: number) {
    return Promise.all([
      async () => this.user = await fetchUser(userId),
      async () => this.posts = await fetchPostsForUser(userId),
    ]);
  }
}
```

2 번의 네트워크 요청이 로드되는 동안 user 와 posts 속성은 null 상태이다. 시점에 따라 두 속성 모두 null 이거나, 하나만 null 이거나, 모두 null 이 아니거나 총 4 가지의 경우가 존재하게 된다. 결국 null 체크가 많아지고 버그를 양산하게 된다.

이런 경우 클래스를 만들기 위한 데이터가 모두 준비된 상태에서 클래스를 생성하는 것이 좋다.

```tsx
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: number) {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId),
    ]);
    return new UserPosts(user, posts);
  }
}
```

이제는 UserPosts 클래스에는 null 이 존재하지 않고, 메서드를 작성하기 쉬워진다.

결론적으로 `strictNullChecks` 옵션은 활성화되면 코드에 많은 오류가 발생하겠지만, `null` 과 관련된 문제점들을 찾을 수 있기에 반드시 필요한 옵션이다. 그리고 `null` 이 가능한 어떤 값이 다른 값의 `null` 여부에 암시적으로 관련되도록 설계해서는 안된다.
