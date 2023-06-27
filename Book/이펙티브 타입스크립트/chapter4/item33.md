# 아이템33. string 타입보다 더 구체적인 타입 사용하기

`string` 타입은 범위는 매우 넓다. 따라서 “x” 나 “y” 같은 한글자도, “Call me…” 로 시작하는 120만 글자의 소설도 모두 `string` 타입이다. string 타입으로 변수를 선언한다면 **더 좁은 타입은 없는지 고민**해봐야 한다.

아래는 음악 앨범을 만들기 위한 타입이다.

```tsx
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // live or studio
}
```

string 이 남발되고 있고, 주석으로 타입 정보를 담고 있기에 잘못된 타입이다. 이처럼 잘못된 값을 설정할 수도 있다.

```tsx
const kindOfBlue: Album = {
  artist: 'artist',
  title: 'title',
  releaseDate: 'August 17th, 1959',
  recordingType: 'Studio',
}
```

`releaseDate` 속성의 값은 YYYY-MM-DD 형식이 아니고, `recordingType` 의 값은 “studio” 가 아닌 “Studio” 지만 문제가 발생하지 않았다.

```tsx
function recordRelease(title: string, date: string) {}
recordRelease(kindOfBlue.releaseDate, kindOfBlue.recordingType);
```

그리고 매개변수의 순서가 잘못들어가도 문제가 발생하지 않는다.

artist 나 title 은 어떤 값이 할당될 지 알 수 없기 때문에 string 타입이 적당하다. 하지만 releaseDate 은 Date 객체만 사용해서 날짜 형식을 제한하는 것이 좋고, recordingType 은 “live” 와 “studio” 2 가지만 사용하는 것이 좋다.

```tsx
type RecordType = 'live' | 'studio';
interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordType;
}

const kindOfBlue: Album = {
  artist: 'artist',
  title: 'title',
  releaseDate: new Date('1959-08-17'),
  recordingType: 'studio',
}
```

이런 방식에는 여러 장점이 있다.

**1. 타입을 명시적으로 정의하여 다른 곳으로 타입정보가 전달되어도 타입 정보가 유지된다.**

**2. 타입을 명시적으로 정의하고 해당 타입이 의미하는 주석을 붙여넣을 수 있다.**

**3. keyof 연산자로 더욱 세밀하게 객체 속성 체크가 가능해진다.**

어떤배열에서 한 필드의 값만 추출하는 함수를 작성한다고 생각해보자. 실제로 언더스코어(_) 에는 pluck 라는 함수가 있다.

```tsx
const albums: Album[] = [
  {
    artist: 'artist',
    title: 'title',
    releaseDate: new Date('1959-08-17'),
    recordingType: 'studio',
  },
  {
    artist: 'artist',
    title: 'title',
    releaseDate: new Date('1959-08-17'),
    recordingType: 'studio',
  }
]

function pluck(records: any[], key: string): any[] {
  return records.map(r => r[key]);
}
console.log(pluck(albums, 'artist')); // ['artist', 'artist']
```

위 코드는 타입체크가 되긴하지만 *`any*[]` 를 return 하기 때문에 정밀하지 못하다. 특히 any 를 return 하는 것은 좋지 않다.

제너릭 타입을 통해 개선할 수 있다. `type *key* = keyof *Album*;` 결과는 `‘artist’ | ‘title’ | ‘releaseDate’ | ‘recordingType’` 이기 때문에 매개변수 key 타입은 `keyof T` 를 사용한다.

```tsx
function pluck<T>(records: T[], key: keyof T): T[keyof T][] {
  return records.map(r => r[key]);
}
console.log(pluck(albums, 'artist'));
```

이 코드는 타입 체커를 통과한다. 함수에 마우스를 올려놓으면 **T 객체 내의 가능한 모든 값의 타입**으로  *`T*[keyof *T*][]` 라는 return type 을 확인할 수 있다. 하지만 key 의 범위가 너무 넓기 때문에 적절한 타입으로 보기 힘들다.

```tsx
const releaseDates = pluck(albums, 'releaseDate'); //type: (string | Date)[]
```

`releaseDates` 의 타입은 `Date[]` 이어야 하지만, `(string | Date)[]` 이 된다.

`keyof T` 는 `string` 에 비해서 훨씬 범위가 좁지만 그래도 여전히 넓다. 범위를 더 좁히기 위해서는 `keyof T` 의 부분집합으로 2 번째 제너릭 타입을 매개변수를 사용해야 한다.

```tsx
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map(r => r[key]);
}

const releaseDates = pluck(albums, 'releaseDate');
const title = pluck(albums, 'title');
const artist = pluck(albums, 'artist');
const recordingType = pluck(albums, 'recordingType');
const recordingDates = pluck(albums, 'recordingDates'); // Argument of type '"recordingDates"' is not assignable to parameter of type 'keyof Album'.ts(2345)
```

매개변수 타입이 정밀해졌기 때문에 Album 의 키에 자동완성 기능이 적용된다.

이처럼 `string` 은 `any` 와 비슷한 문제점을 가지고 있다. 따라서 잘못 사용되면 무효한 값을 허용하고 타입간의 관계도 감춰버린다. 타입스크립트에서 string 의 부분집합을 정의할 수 있는 기능은 자바스크립트 코드에 타입 안정성을 크게 높여준다.

만약 객체 속성의 이름을 매개변수로 받아야 한다면 `string` 보다는 `keyof T` 를 사용하는 것이 좋다.
