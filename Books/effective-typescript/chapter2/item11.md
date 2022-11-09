# 아이템11. 잉여 속성 체크의 한계 인지하기

타입이 선언된 변수에 객체 리터널을 할당할 때, 타입스크립트는 **해당 타입의 속성이 있는지**, 그리고 **그 외의 속성은 없는지**를 확인한다. 그리고 이것을 잉여 속성 체크라고 한다.

```tsx
// EX1
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const room: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // TS2322: Type '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' is not assignable to type 'Room'.   Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'.
}
```

구조적 타이핑 관점에서는 오류가 발생하지 않아야 한다. 반대로 room 타입의 변수에 임시 변수를 도입해보면 문제가 없는 것을 확인할 수 있다.

```tsx
// EX2
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
}

const room: Room = obj;
```

obj 타입은 `{numDoors: number, ceilingHeightFt: number, elephant: string}` 로 추론되기에 **obj 타입은 Room 타입의 부분 집합을 포함**하므로, Room 타입에 할당가능하며 타입체커에서도 통과된다.

`EX1`에서는 구조적 타이핑의 중요한 종류의 오류를 잡을 수 있도록 **잉여 속성 체크라는 과정이 추가**되었다. 그러나 잉여 속성 체크도 조건에 따라 동작하지 않는다는 한계가 있고, 할당 가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수도 있다. **잉여 속성 체크와 할당 가능 검사는 별도**라는 것을 기억하고 있어야 한다.

타입스크립트는 런타임에 오류를 표시하는 것뿐만이 아니라, 의도와 다르게 작성된 코드까지 찾으려 한다.

```java
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    // ...
  }
  // ...
}

createWindow({
  title: 'Spider Solitaire',
  darkmode: true, // TS2345: Argument of type '{ title: string; darkmode: boolean; }' is not assignable to parameter of type 'Options'.   Object literal may only specify known properties, but 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?
})
```

하지만 `Options` 타입의 범위는 상당히 넓다. darkMode 속성에 `boolean` 타입이 아닌 다른 값이 지정된 경우를 제외하면, `**string` 타입의 title 속성과 또 다른 속성을 가지고 있는 모든 객체는 `Options` 타입범위에 속한다**.

```tsx
interface Options {
  title: string;
  darkMode?: boolean;
}

const o1: Options = document;
const o2: Options = new HTMLAnchorElement;
```

`document`와 `HTMLAnchorElement`의 인스턴스 모두 string 타입인 title 속성을 가지고 있기 때문에 할당문(`o1, o2`)은 정상이 된다.

**잉여 속성 체크를 사용한다면 기본적으로 타입 시스템의 구조적인 본질을 해치지 않고, 객체 리터널에 알 수 없는 속성을 허용하지 않으면서 위의 문제점들을 방지**할 수 있다.

```tsx
const o1: Options = { darkmode: true, title: 'Sko Free' }; // 오류
// TS2322: Type '{ darkmode: boolean; title: string; }' is not assignable to type 'Options'.   Object literal may only specify known properties, but 'darkmode' does not exist in type 'Options'. Did you mean to write 'darkMode'?

const intermediate = { darkmode: true, title: 'Sko Free' }; // 정상
const o2: Options = intermediate;
```

변수에 할당되는 값이 첫번째 케이스는 **객체 리터널이기에 오류가 발생**하지만, 두번째 케이스는 **객체 리터널이 아니기 때문에 잉여 속성 체크가 적용되지 않고 오류가 발생하지 않는다**.

[타입 스크립트에서는 신선도(Freshness)](https://radlohead.gitbook.io/typescript-deep-dive/type-system/freshness)라는 개념을 제공한다. 모든 객체 리터널은 초기에 `fresh` 하다고 간주되며, 1) 타입 단언(type assertion)을 하거나, 2) 타입 추론에 의해 object literal의 타입이 확장되면 `freshness`가 사라지게 된다. 특정 변수에 객체 리터널을 할당할 경우 2 가지중 한가지가 발생하게 되므로 `freshness` 가 사라지게 되고, **함수에 인자로 객체 리터널을 할당할 경우 `fresh` 한 상태로 전달**되게 된다.

```tsx
// 출처: https://toss.tech/article/typescript-type-compatibility
type Food = {
  protein: number;
  carbohydrates: number;
  fat: number;
}

type Burger = Food & {
  burgerBrand: string;
}

function calculateCalorie(food: Food){
  return food.protein * 4
    + food.carbohydrates * 4
    + food.fat * 9
}

const burger = {
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
}

const calorie = calculateCalorie(burger)

const calorie2 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: '버거킹'
})
```

잉여 속성 체크는 타입 단언문을 사용할 때도 적용되지 않는다.

```tsx
const o: Options = { darkmode: true, title: 'Sko Free' } as Options; // 정상
```

이것이 명백하게 단언문보다 선언문을 사용해야 되는 이유이다.

약한(weak) 타입에서도 비슷한 체크가 작동한다.

```tsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true };
const op: LineChartOptions = opts; // TS2559: Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.
```

구조적 관점에서 LineChartOptions 타입은 모든 속성이 선택적인 약한 타입이므로 **타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행**한다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타같은 실수를 바로 잡는데 효과적이다.

선택적 필드를 포함하는 Options 와 같은 타입에는 유용하지만, 적용범위도 제한적이고 오직 객체 리터널에만 적용된다.
