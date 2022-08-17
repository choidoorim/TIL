# TypeORM Transaction
기본적으로 TypeORM 에서 트랜잭션은 typeorm 모듈의 Connection 또는 EntityManager 를 사용하게 됩니다.

트랜잭션에서 실행하려는 로직들은 반드시 모두 콜백을 통해 실행되어야 합니다.
### 1. getManager 을 사용하는 방법 - [공식문서](https://orkhan.gitbook.io/typeorm/docs/transactions)
transactionalEntityManager 와 쿼리 수행에 필요한 값을 인자로 Repository 에 전달하고, 
Repository 에서는 transactionalEntityManager 를 통해서 DB 작업을 진행합니다.

#### xxx.service.ts
```typescript
import {getManager} from "typeorm";
export class UserService {
    constructor(private usersRepository: UserRepository) {}
    //...
    async doUserRegistration(registerUserInfo){
        await getManager().transaction(async transactionalEntityManager => {
            // 트랜잭션 실행 로직
            await this.usersRepository.doUserRegistration(
                transactionalEntityManager,
                registerUserInfo,
            );
            // ...
        }).catch((err) => {
            throw err
        });
    }   
}
```

만약 별도 Repository 파일을 만들어 사용하지 않는다면 아래와 같이 Service 로직에서 바로 transactionalEntityManager 를 통해 로직을 수행하면 됩니다.

#### Repository (X) -  xxx.service.ts
```typescript
import {getManager} from "typeorm";
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

export calss UserService {
    constructor(
        @InjectRepository(User)
        private usersRepository: Repository<User>,
    ) {}
    // ...
    async doUserRegistration(registerUserInfo){
        const createUserInfo: User[] = this.usersRepository.create(registerUserInfo);
        await getManager().transaction(async transactionalEntityManager => {
            // 트랜잭션 실행 로직
            await transactionalEntityManager.save(createUserInfo);
            // ...
        }).catch((err) => {
            throw err
        });
    }   
}
```

### 2. queryRunner 를 사용하는 방법 - [공식문서](https://orkhan.gitbook.io/typeorm/docs/query-runner)
commit, rollback, release 등의 transaction 상태를 수동으로 제어할 수 있고, 트랜잭션 수행로직이 직접적으로 보이기에 가독성이 좋다는 장점이 있는 것 같습니다.


#### xxx.service.ts
```typescript
async doUserRegistration(registerUserInfo: CreateUserDTO): Promise<User[]> {
    const queryRunner = await getConnection().createQueryRunner();
    await queryRunner.startTransaction();
    try {
      // 트랜잭션 실행 로직
      const result = await this.usersRepository.doUserRegistration(
        queryRunner.manager,
        registerUserInfo,
      );
      // ...
      await queryRunner.commitTransaction();
      return result;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw new NotFoundException(`Failed SignUp - ${error}`);
    } finally {
      await queryRunner.release();
    }
  }
```

### Repository
Repository 에서는 Service 에서 인자로 전달받은 EntityManager 를 통해 쿼리 로직을 수행합니다.

#### xxx.repository.ts
```typescript
import {
    EntityRepository,
    Repository,
    TransactionManager,
    EntityManager,
} from 'typeorm';
import { User } from './user.entity';

export class UserRepository extends Repository<User> {
    public async doUserRegistration(
        @TransactionManager() transactionManager: EntityManager,
        registerUserInfo,
    ) {
        const createUserInfo: User[] = this.create(registerUserInfo);

        return await transactionManager.save(createUserInfo);
    }
}
```

개인적으로는 QueryRunner 를 사용하는 방법이 코드가 직관적으로 들어오기 때문에 트랜잭션 처리는 QueryRunner 를 사용하고 있습니다.
