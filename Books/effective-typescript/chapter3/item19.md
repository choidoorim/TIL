# 아이템19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트에서 많은 타입 구문은 사실 불필요하다.

```tsx
let x: number = 12;
```

사실 아래와 같이해도 충분하다.

```tsx
let x = 12; // let x: number;
```

편집기에서 마우스를 올려보면 타입이 이미 number 로 추론되어 있는 것을 확인할 수 있다. 타입추론이 된다면 명시적 타입 구문은 오히려 방해가 될 뿐이다.

```tsx
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  }
  died: {
    where: string;
  }
} = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1997',
  },
  died: {
    where: 'Battle Creek, MI',
  }
}
```

이 코드는 타입을 생략하고 사용해도 된다.

```tsx
const person = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1997',
  },
  died: {
    where: 'Battle Creek, MI',
  }
}
```

이처럼 추가로 타입을 작성하는 것은 거추장스러울 뿐이다.

배열도 객체와 마찬가지이다. 타입스크립트는 입력을 받아 연산을 하는 함수가 어떤 타입을 반환하는지 정확히 알고 있다.

```tsx
function square(nums: number[]) {
  return nums.map(x => x * x);
}

const squares = square([1, 2, 3, 4]); // const squares: number[];
```

타입스크립트는 생각보다 더 정확하게 추론을 한다.

```tsx
const axis1: string = 'x';
const axis2 = 'y'; // const axis2: "y"
```

`axis2` 변수를 `string` 으로 예상하기 쉽지만 **실제로 타입스크립트는 `y` 로 추론**한다.
