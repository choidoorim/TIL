# 아이템27. 함수형 기법과 라이브러리로 타입 흐름 유지하기

Python, C, Java 등에서 제공해주는 표준 라이브러리가 Javascript 에서는 포함되어 있지 않다.

자바스크립트 라이브러리들을 타입스크립트와 함께 사용하면 더욱 더 빛을 발한다. 타입 정보가 그대로 유지되면서 타입 흐름(flow)이 계속전달되게 하기 때문이다.

아래는 csv 파일을 파싱하는 예제이다.

```tsx
const csvData = '...';
const rawRows = csvData.split('\n');
const headers = rawRows[0].split(',');

const rows = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(',').forEach((val, j) => {
    row[headers[j]] = val;
  });
  return row;
});

const rows = rawRows
  .slice(1)
  .map((rowStr) =>
    rowStr
      .split(',')
      .reduce((row, val, i) => ((row[headers[i]] = val), row), {}),
  );
```

reduce 를 사용함으로써 코드를 절약했다. 하지만 보는 사람이 더 복잡하게 느껴질 수 있다.

```tsx
import * as _ from 'lodash';

const rows = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(',')));
```

키와 값 배열로 취합해서 객체로 만들어주는 `lodash` 의 `zipObject` 함수를 사용하면 코드를 더욱 짧게 만들 수 있다.

자바스크립트에 서드파티 라이브러리 종속성을 추가할 때는 신중해야 한다. 만약 서드파티 라이브러리 기반으로 코드를 짧게 만드는 시간이 많이 소요된다면 사용하지 않는게 더 좋기 때문이다.

그러나 같은 코드를 타입스크립트로 작성하면 서드파티 라이브러리를 사용하는 것이 무조건 유리하다. 타입 정보를 참고하면서 작업할 수 있기 때문이다.

예를 들어 NBA 선수들의 명단을 가지고 있다고 가정해보자.

```tsx
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}
type Rosters = Record<string, BasketballPlayer[]>;

declare const rosters: {[team: string]: BasketballPlayer[]};
declare const rosters: Rosters;
```

루프를 사용해서 list 목록을 만들기 위해서는 배열에 `concat` 을 사용해야 한다.

```tsx
let allPlayers = [];
for (const players of Object.values(rosters)) {
	// allPlayers type: any[]
  allPlayers = allPlayers.concat(players);
}
```

이 코드는 동작에는 문제없지만 타입 체크가 되지 않는다. 해결하기 위해서는 allPlayers 변수에 타입 선언을 하면 된다.

```tsx
let allPlayers: BasketballPlayer[] = [];
```

하지만 `Array.prototype.flat` 을 사용하는 것이 더 좋은 해법이다.

```tsx
const allPlayers = Object.values(rosters).flat();
```

flat 메서드는 다차원 배열을 평탄화하게 해준다. 타입은 `T[][] → T[]`  같은 형태이다.

팀별로 연봉순으로 정렬해서 최고 연봉 선수의 명단을 만드는 예제이다.

```tsx
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}
const rosters: { [team: string]: BasketballPlayer[] } = {
  team1: [
    {
      name: 'name1',
      team: 'team1',
      salary: 3,
    },
    {
      name: 'name2',
      team: 'team1',
      salary: 4,
    },
  ],
  team2: [
    {
      name: 'name1',
      team: 'team2',
      salary: 1,
    },
    {
      name: 'name2',
      team: 'team2',
      salary: 10,
    },
  ],
};

const allPlayers = Object.values(rosters).flat();
const teamToPlayers: { [team: string]: BasketballPlayer[] } = {};
for (const player of allPlayers) {
  const { team } = player;
  teamToPlayers[team] = teamToPlayers[team] || [];
  teamToPlayers[team].push(player);
}

// 팀별 연봉 순으로 정렬
for (const players of Object.values(teamToPlayers)) {
  players.sort((a, b) => b.salary - a.salary);
}

// 팀내 최고 연봉 선수 추출
const bestPaid = Object.values(teamToPlayers).map((players) => players[0]);
// 연봉 순으로 정렬
bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary);
```

위 코드를 로대시를 사용해서 변경하면 이용하면 아래와 같다.

```tsx
const bestPaid = _(allPlayers)
  .groupBy((player) => player.team)
  .mapValues((player) => _.maxBy(player, (p) => p.salary))
  .values()
  .sortBy((p) => -p.salary)
  .value();
console.log(bestPaid);
```

체인을 사용하면 연산자의 등장 순서와 실행 순서가 동일하게 된다. `_(v).a().b().c().value()`

`_(v)` 에서 값을 래핑하고 `.value()` 에서 언래핑한다.

내장된 `map` 함수 대신에 `lodash` 의 `_.map` 을 사용하는 이유가 있다. 콜백을 전달하는 대신 속성의 이름을 전달할 수 있기 때문이다.

```tsx
const namesA = allPlayers.map(player => player.name);
const namesB = _.map(allPlayers, player => player.name);
const namesC = _.map(allPlayers, 'name');
```

함수 호출 시 전달된 매개변수 값을 건드리지 않고 매번 새로운 값을 반환하여, **새로운 타입으로 안전하게 반환**할 수 있다.
