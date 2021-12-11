# ORM N + 1 문제
## N + 1 문제가 뭘까?

ORM 의 연관 관계에서 발생하는 문제로, 
연관 관계가 설정된 엔티티를 조회할 경우에 **조회된 데이터 갯수(N) 만큼 연관관계의 조회 쿼리가 추가로 발생**하여 데이터를 읽어오게 되는 것입니다.

예를 들어 사용자는 여러개의 상품을 가지고 있을 수 있기 때문에 ```User``` 와 ```Product```  엔티티가 1 : N 관계로 설정되어 있다고 가정해봅시다.    
특정 유저를 조회 했을 때 (SELECT 1번),
해당 유저와 관련된 상품을 N 번 더 조회합니다(SELECT N번).

위의 문제에 대한 해결 방법을 알아보기 전에 관련된 중요한 개념인 **Eager Loading** 과 **Lazy Loading** 에 대해서 알아봅시다.

## Eager Loading 이란?
Eager Loading 이란 **즉시 로딩** 으로 로딩시 참조해야 할 데이터들을 **전부 미리 가져옵니다**.    
연관 된 모든 리소스들을 한 번에 가져올 수 있다는 장점이 있지만, 리소스를 전부 사용하지 않는다면 낭비될 수 있고, 초기 로딩 속도가 느려질 수 있다는 특징이 있습니다.

아래처럼 User 와 Board 가 1: N 관계로 이루어져있습니다.

![1-N](https://user-images.githubusercontent.com/63203480/145675344-737e6f03-c9ff-4768-abdf-f739cd5637da.PNG)

#### user.entity.ts
```typescript
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToMany,
} from 'typeorm';
import { Board } from "../board/board.entity";

@Entity({ name: 'user' })
export class User {
    @PrimaryGeneratedColumn()
    idx: number;

    @Column()
    name: string;

    @OneToMany((type) => Board, (Board) => Board.user)
    board: Board[];
}
```

#### board.entity.ts
```typescript
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    ManyToOne,
    JoinColumn,
} from 'typeorm';
import { User } from "../user/user.entity";

@Entity({ name: 'board' })
export class Board {
    @PrimaryGeneratedColumn()
    idx: number;

    @Column()
    title: string;

    @Column()
    contents: string;

    @ManyToOne(() => User, (user) => user.board)
    user: User;
}
```

기존 Spring 의 JPA 는 ```OneToMany``` 의 default fetch Type 은 Lazy Loading 이며 ManyToOne 의 default fetch Type 은 Eager 이기 때문에 N + 1 문제를 바로 겪을 수 있습니다.     
하지만 TypeORM 은 조금 다릅니다.

아래 사진에서 Relation Option 코드를 확인해보면 TypeORM 은 기본적으로 어떤 종류의 관계에서도 기본적으로 fetch type 을 적용하지 않는 것을 확인할 수 있습니다.
즉, 기본적으로 명시하지 않는다면 Eager 도 Lazy 도 아니라는 것입니다.

![relation](https://user-images.githubusercontent.com/63203480/145675604-831bec78-2d69-4fee-bb65-7743519c0560.PNG)

fetch Type 에 ```eager``` 을 적용한 후 ```findOne()``` 메서드를 사용하여 조회를 해보면 어떻게 될까요?

```typescript
    // ...
    @OneToMany((type) => Board, (Board) => Board.user, {
        eager: true
    })
    board: Board[];
    // ...
```

```typescript
    //...
    async findOne(userIdx: number): Promise<User> {
        return await this.usersRepository.findOne(userIdx);
    }
```

![1](https://user-images.githubusercontent.com/63203480/145676387-d873b6ca-bca5-4680-be20-e04477c4fefb.PNG)

```
query: SELECT `User`.`idx` AS `User_idx`, `User`.`name` AS `User_name`, `User_board`.`idx` AS `User_board_idx`, `User_board`.`title` AS `User_board_title`, `User_board`.`contents` AS `User_board_contents`, `User_board`.`user
Idx` AS `User_board_userIdx` FROM `user` `User` LEFT JOIN `board` `User_board` ON `User_board`.`userIdx`=`User`.`idx` WHERE `User`.`idx` IN (?) -- PARAMETERS: [1]
```

위와 같이 연관된 데이터가 전부 조회되는 것을 확인할 수 있습니다.   
이처럼 연관된 데이터들을 한 번에 가져오는 것을 Eager Loading 이라고 합니다. TypeORM 에서는 Eager Loading 을 사용할 때 주의사항이 있습니다.

### 주의사항

1. Entity 의 Eager Loading 설정은 find 메서드를 사용할 때만 작동합니다.
2. 단순히 QueryBuilder 를 사용할 때는 적용되지 않으며, Eager Loading 의 효과를 보고 싶으면 leftjoinAndSelect 를 사용하면 됩니다.
3. Eager 설정은 관계를 한쪽에서만 설정이 가능하고, 양쪽에서 true 로 할 수는 없습니다. 만약 사용할 경우 에러가 발생합니다.
   ![1](https://user-images.githubusercontent.com/63203480/145676631-b4c41fe0-8065-4f70-b895-2fcc5f4d4ddc.PNG)    
   연관된 User 와 Board 가 서로 데이터를 가져오는 무한 참조현상 때문입니다.
   
Eager Loading 은 join 을 통해 연관된 데이터를 한 번에 가져오는 장점이 있지만 단점도 존재합니다.

### 단점
- 긴 초기 로딩 시간(연관된 데이터를 한 번에 가져오기 때문에)
- 불필요한 데이터들도 조회되는 상황이 발생합니다. 즉, 프론트에서 필요 없는 데이터까지 받아야하는 오버 패칭이 발생한다는 것입니다.
- 연관된 Entity 가 많을수록 join 이 발생하게 되고, 이러한 join 때문에 성능 저하가 일어날 수 있습니다.

## Lazy Loading 이란?
지연 로딩이라고하며, Eager Loading 과 반대로 **필요한 순간에만 데이터를 가져옵니다**.   
아래와 같이 설정이 가능합니다.

```typescript
    // ...
    @OneToMany((type) => Board, (Board) => Board.user, {
        lazy: true
    })
    board: Board[];
    // ...
```

Lazy Loading 을 적용하여 조회를 하면 어떻게 될까요? 

![1](https://user-images.githubusercontent.com/63203480/145676923-10e5d336-9a1a-473a-aa84-d30e5cfa4c4b.PNG)

Board 가 필요하다고 요청하지 않았기 때문에 연관된 데이터가 조회되지 않았습니다.
Lazy Loading 은 특성상 필요한 경우게만 해당 데이터를 가져오기 때문입니다. 따라서 User 데이터만 조회된 것입니다.

그렇다면 Board 데이터가 필요해서 요청해야 할 경우에는 어떻게 해야 할까요?

```typescript
    // ...
    async findOne(userIdx: number): Promise<User> {
        const users: User = await this.usersRepository.findOne(1);
        const boards: Board[] = await users.board;   // Board 데이터를 요청
        return users;
    }
    // ...
```

위와 같이 Board 데이터를 요청하는 코드를 추가해줘야 합니다.

![1](https://user-images.githubusercontent.com/63203480/145677193-5beb6fac-97b2-495b-b172-c27bd38cec49.PNG)

추가 후 조회를 하게 되면 위와 같은 결과가 나옵니다. TypeORM 에서 Lazy Loading 을 사용하기 위해서는 반드시 비동기 처리를 해줘야 한다고 합니다. 따라서 ```await``` 를 통해 요청을 한 것입니다.     
쿼리를 확인해보면 join 이 아닌, select 쿼리가 한 번더 출력된 것을 확인할 수 있습니다.

```
query: SELECT `User`.`idx` AS `User_idx`, `User`.`name` AS `User_name` FROM `user` `User` WHERE `User`.`idx` IN (?) -- PARAMETERS: [1]
query: SELECT `board`.`idx` AS `board_idx`, `board`.`title` AS `board_title`, `board`.`contents` AS `board_contents`, `board`.`userIdx` AS `board_userIdx` FROM `board` `board` WHERE `board`.`userIdx` IN (?) -- PARAMETERS: [1
]
```

Eager Loading 에서는 join 을 사용하여 1 번의 쿼리로 해당 명령을 수행하지만, Lazy Loading 의 경우 추가적인 조회 쿼리가 발생하는 것을 확인할 수 있습니다.

Lazy Loading 은 Eager Loading 에 비해 쿼리가 많이 발생한다는 단점이 있지만, 장점도 있습니다.

### 장점
1. 초기 로딩 시간을 줄일 수 있습니다.(연관된 데이터를 필요할 때 요청하기 때문)
2. 자원 소비를 Eager Loading 을 사용했을 때에 비해 줄일 수 있습니다.

## N + 1 문제 해결 방법
관련된 개념을 살펴봤으니 TypeORM 에서 ```N + 1``` 문제를 어떻게 해결할까요?

Lazy Loading 을 적용 후 ```find``` 메서드를 사용했을 경우 추가적인 쿼리가 발생한 것을 확인할 수 있었습니다. 
이 문제가 바로 **N + 1** 문제 입니다. 1 번의 쿼리를 사용해서 N 개의 데이터를 가져오는데, 관련된 데이터들을 얻기 위해 추가적으로 N 번의 쿼리가 실행되는 문제입니다. 
그래서 총 N + 1 번의 쿼리가 발생되는 문제입니다. 

예를 들어 **User Table 에 10 건의 데이터**가 있다고 가정을 합시다. 
Lazy Loading 을 사용한다면 1 번의 쿼리로 User Table 을 조회하고, 조회된 User 의 수만큼 Board 테이블에서 User 와 관계를 맺고 있는 연관된 데이터를 찾기위해 10 번의 추가적인 쿼리가 발생하게 됩니다.

TypeORM 에서는 이러한 N + 1 문제를 **find 메서드에 relations 옵션을 적용하거나, QueryBuilder 를 사용해서 해결**한다고 합니다.

### Relations 옵션

```typescript
    // ...
    async findAll(): Promise<User[]> {
        const users: User[] = await this.usersRepository.find({
            relations: ['board']
        });
        for (const user of await users) {
            const boards = await user.board;
        }
        return users;
    }
    // ...
```

위와 같이 relations 옵션을 설정하게 될 경우, fetch type 을 lazy 로 설정했지만 조회 시 join 쿼리가 실행되는 것을 확인할 수 있습니다.

```
query: SELECT `User`.`idx` AS `User_idx`, `User`.`name` AS `User_name`, `User__board`.`idx` AS `User__board_i
dx`, `User__board`.`title` AS `User__board_title`, `User__board`.`contents` AS `User__board_contents`, `User_
_board`.`userIdx` AS `User__board_userIdx` FROM `user` `User` LEFT JOIN `board` `User__board` ON `User__board
`.`userIdx`=`User`.`idx`
```

### QueryBuilder
QueryBuilder 는 기본적으로 우리가 작성하는 Raw Query 와 비슷합니다. 

```typescript
    async findAll(): Promise<User[]> {
    return await this.usersRepository.createQueryBuilder('user')
        .leftJoinAndSelect('user.board', 'board')
        .getMany();
}
```

위와 같이 작성 시 동일한 결과를 얻을 수 있습니다.

```
query: SELECT `user`.`idx` AS `user_idx`, `user`.`name` AS `user_name`, `board`.`idx` AS `board_idx`, `board`.`title` AS `board_title`, `board`.`contents` AS `board_contents`, `board`.`userIdx` AS `board_userIdx` FROM `user`
 `user` LEFT JOIN `board` `board` ON `board`.`userIdx`=`user`.`idx`
```


참고    
https://velog.io/@ypd03008/TypeORM-N1-%EB%AC%B8%EC%A0%9C       
https://tristy.tistory.com/m/36
