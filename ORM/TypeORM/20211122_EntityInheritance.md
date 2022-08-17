## NestJS, TypeORM - Entity inheritance
프로젝트를 진행하면서 entity 를 개발하던 도중 entity 별로 중복되는 컬럼들(idx, createdAt, updatedAt, status...)이 있었다.
이를 해결할 방법을 찾아보니 Entity inheritance(Entity 상속) 이라는 것이 있었다.

### 해결 방법 1 - Concrete Table Inheritance
공통 되는 필드를 추상 클래스를 두고 상속을 통해 사용하는 방식이다.
```common.entity.ts``` 파일을 만든 후, 다른 Entity 들을 개발할 때는 상속해서 사용할려고 한다.

#### common.entity.ts
```typescript
import {
  Column,
  CreateDateColumn,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from 'typeorm';

export abstract class Common {
  @PrimaryGeneratedColumn()
  idx: number;

  @CreateDateColumn({
    type: 'timestamp',
  })
  createdAt: Date;

  @UpdateDateColumn({
    type: 'timestamp',
  })
  updatedAt: Date;

  @Column({ default: false })
  status: boolean;
}
```

#### user.entity.ts
```typescript
import { Common } from './common.entity';
//...

@Entity({ name: 'user' })
export class User extends Common {
    // ...
}
```

### 해결 방법 2 - Single Table Inheritance
```typeorm``` 모듈의 ```@TableInheritance, @ChildEntity``` 를  사용하는 방법이다.

#### common.entity.ts
```typescript
@Entity()
@TableInheritance({ column: { type: "varchar", name: "type" } })
export class Common {
    @PrimaryGeneratedColumn()
    idx: number;

    @CreateDateColumn({
        type: 'timestamp',
    })
    createdAt: Date;

    @UpdateDateColumn({
        type: 'timestamp',
    })
    updatedAt: Date;

    @Column({ default: false })
    status: boolean;
}
```

#### user.entity.ts
```typescript
@ChildEntity({ name: 'user' })
export class User extends Common {
    // ...
}
```

확실히 위의 2가지의 방법처럼 상속을 통해 Entity 를 개발하니 공통으로 사용되는 코드, 컬럼들을 재사용할 수 있어서 편리하면서 좋은 것 같다.
