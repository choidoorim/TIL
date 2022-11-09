## 5. any 타입 지양하기

타입스크립트의 타입시스템은 **점진적이고 선택적**이다. 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입체커를 해제할 수 있기 때문에 선택적이다. 이 기능들의 핵심은 any 이다.

any를 사용하게 된다면 그 위험성을 알고 있어야 한다.

### any 타입에는 타입 안전성이 없다

```tsx
let age: number;
age = '12' as any;
age += 1;
console.log(age); // 121
console.log(typeof age); // string
```

age 가 number 타입으로 선언되었지만 as any 를 사용해서 string 타입을 할당했다. 런타임 시에는 문제가 발생하지 않는다. 그리고 타입 체커는 선언에 따라 number 타입으로 판단할 것이고 혼란스러운 상황이 발생한다.

### any는 함수 시그니처(번역: Contract)를 무시한다

호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입을 반환해야 하지만 `any` 를 사용하면 이런 약속을 어기는 상황이 발생한다.

```tsx
function calculateAge(birth: Date): number {
  return birth.getDate();
}

let birthDate: any = '1990-01-19';
calculateAge(birthDate);
```

위의 코드는 타입체커에서 문제로 인식하지 못한다. `calculateAge` 함수의 조건을 무시하게 된다. 이렇게 타입이 불일치하는 상황이 많아지면 다른 곳에서 문제를 일으키는 상황이 많이 발생하게 될 것이다.

### any 타입에는 언어 서비스가 적용되지 않는다

타입이 있다면 언어 서비스는 자동완성 기능을 제공해준다. 하지만 any 타입을 사용하면 아무런 도움도 받을 수 없다.

```tsx
interface Person {
  name: string;
  age: number;
}

const person1: Person = { name: 'choi', age: 11 }
const person2: any = { name: 'choi', age: 11 }
```

위의 코드에서 Person 타입에 name 을 firstName 으로 바꾸기 위해서 편집기의 Rename 기능을 사용하면 any 타입의 person2 는 아무런 기능도 제공받지 못하게 된다.

![Untitled 1](https://user-images.githubusercontent.com/63203480/199016144-7048061c-6905-42cf-b6c6-c11e86b3f1c1.png)

### any 타입은 코드 리팩터링 때 코드를 감춥니다

```tsx
interface ComponentProps {
  onSelectItem: (item: any) => void;
}

function renderSelector(props: ComponentProps): null {
  return null;
}

let selectedId: number = 1;
function handleSelectItem(item: any) {
  selectedId = item.id;
}

renderSelector({onSelectItem: handleSelectItem});
```

`item` 의 타입을 알 수 없는 상황이기에 `any` 타입으로 선언을 했다. 이 상황에서 `ComponentProps` 타입을 아래와 바꾸면 어떻게 될까?

```tsx
interface ComponentProps {
  onSelectItem: (id: number) => void;
}
```

`handleSelectItem` 메서드의 매개변수는 `any` 타입을 받고 있기에 문제가 발생하지 않는다. 만약 `any` 타입을 사용하지 않았다면 타입 체커가 문제를 발견했을 것이다

### any는 타입 설계를 감춰버립니다

객체를 정의할 때, 상태 객체의 설계를 감춰버리기 때문에 문제가 생긴다.

- 객체의 상태: 특정 시점에 객체가 가지고 있는 정보의 집합으로 객체의 구조적 특징

깔끔하고 정확하고 명료한 코드 작성을 위해 제대로된 타입설계는 필수이다. `any` 타입을 사용하면 타입의 설계가 불분명해진다. 협력하기 위해서는 설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋다.

### any 는 타입시스템의 신뢰도를 떨어트린다

사람은 항상 실수하기에 타입 체커가 실수를 잡아주고 코드의 신뢰도를 높여준다. any 타입을 사용하지 않는다면 **런타임에서 발견될 오류를 미리 잡고 신뢰도를 높일 수 있다**.

그리고 any 타입은 자바스크립트를 사용하는 것보다 더 힘든 상황을 만들 수 있다.

하지만 어쩔 수 없이 any 타입을 사용해야만 하는 경우도 존재한다.
