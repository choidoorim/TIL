# 아이템36. 해당 분야의 용어로 타입 이름 짓기

```
컴퓨터 과학에서 어려운 일은 단 두가지 뿐이다. 캐시 무효화와 이름 짓기.
- 필 칼튼
```

타입의 이름짓기도 타입 설계에 중요한 부분이다. 타입, 속성, 변수의 이름을 잘 짓는다면 의도를 명확히 하고 코드와 타입의 추상화 수준을 높여준다.

동물 데이터베이스를 구축한다고 해보자.

```tsx
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

const leopard: Animal = {
  name: 'Snow Leopard',
  endangered: false,
  habitat: 'tundra',
}
```

이 코드에는 몇 가지 문제가 있다.

1. name 은 일반적인 용어이다. 동물의 종 이름인지 일반적인 명칭인지 알 수가 없다.
2. endangered 속성이 멸종 위기를 표현하기 위해 boolean 을 사용하는 것이 이상하다. 이미 멸종된 동물을 true 로 해야하는지 판단할 수 없다. 그리고 속성을 멸종위기 또는 멸종으로 생각할 수 있다.
3. 서식지를 나타내는 habitat 속성은 범위가 너무 넓고, 뜻 자체가 너무 모호하다.
4. 객체의 변수명이 leopard 이지만 속성명은 Snow Leopard 이다. 객체의 이름과 속성의 name 이 다른 의도로 사용되는 것인지 불분명하다.

반면 아래 코드의 타입 선언은 의미가 분명하다.

```tsx
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimates[];
}

type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimates = | 'Af' | 'Am' | 'As' | 'Aw' | 'ET'| 'EF'| 'Dfd';

const snowLeopard: Animal = {
  commonName: 'Snow leopard',
  genus: 'Panthera',
  species: ' Uncia',
  status: 'VU',
  climates: ['ET', 'EF', 'Dfd' ],
}
```

이전 코드보다 세 가지를 개선했다.

1. `name` 은 `commonName`, `genus`, `species` 등 더 구체적인 용어로 대체했다.
2. `endangered` 는 동물 보호 등급에 대한 IUCN 표준 분류 체계인 `ConservationStatus` 타입의 `status` 로 변경했다.
3. `habitat` 은 기후를 뜻하는 `climates` 로 변경됐고, 쾨펜 기후 분류를 사용한다.

코드를 변경해서 데이터를 훨씬 더 명확하게 표현하고 있다. 따라서 코드에 대한 정보를 알기 위해서 작성한 사람을 찾을 필요도 없어졌다. 코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 있다. 자체적으로 용어를 만들기보다는 해당 분야에서 사용하고 있는 용어를 사용해야 한다.

타입, 속성, 변수에 이름을 붙일 때 명심해야 할 3 가지 규칙이 있다.

1. 동일한 의미를 나타낼 때는 같은 용어를 사용해야 한다. 동의어는 글을 읽을 때 좋을 수 있지만, 코드에서는 좋지 않다.
2. data, info, thing, item, object, entity 와 같은 모호하고 의미없는 이름을 피해야 한다.
3. 이름을 지을 때 포함된 내용이나 계산 방식이 아니라 **데이터 자체가 무엇인지 고려**해야 한다.
