# NestJS, TypeORM 비밀번호 단방향 암호화
비밀번호가 그대로 데이터베이스에 저장되면 안되기 때문에 단방향 암호화를 통해 암호화된 비밀번호를 저장해야합니다.   
암호화를 도와주는 모듈인 Bcrypt 를 설치해서 사용할 것입니다.

```
npm install bcrypt
npm install @types/bcrypt 
```

그리고 entity 에서는 ```@Beforeinsert()``` 데코레이터를 사용합니다. 데이터가 insert 되기 전에 실행할 수 있도록 도와주는 데코레이터입니다.
- entity 란 NestJS 에서 정의하는 데이터베이스 모델이다.

### 1.Entity(xxx.entity.ts) 에 @Beforeinsert 추가하기
```typescript
import { 
    Entity,
    Column, 
    PrimaryGeneratedColumn,
    BeforeInsert
} from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity({name: 'user'})
export class User{
    @PrimaryGeneratedColumn()
    idx: number;

    @Column({default:false})
    status: boolean;

    @Column({ length: 45 })
    email: string;

    @Column({type: "text"})
    password: string;
    
    
    @BeforeInsert()
    async setPassword(password: string) {
        const salt = await bcrypt.genSalt();
        this.password = await bcrypt.hash(password || this.password, salt);
    }
}
```

기존의 password 와 암호화에 사용될 salt 를 만들어서 hashing 을 합니다.

Service 단에서 회원가입에 대한 비지니스 로직을 처리합니다.

```typescript
@Injectable()
export class UserService {
    constructor(
        @InjectRepository(User) private usersRepository: Repository<User>,
    ) {}

    async doUserRegistration(registerUserInfo: CreateUserDTO): Promise<User> {
        const queryRunner = await getConnection().createQueryRunner();
        await queryRunner.startTransaction()
        try {
            const user: User = await this.usersRepository.create(registerUserInfo);

            const result: User = await this.usersRepository.save(user);

            await queryRunner.commitTransaction();
            return result;
        } catch (error) {
            await queryRunner.rollbackTransaction();
            throw new NotFoundException(`Failed SignUp ${error}`);
        } finally {
            await queryRunner.release();
        }
    };
}
```

TypeORM 의 ```getConnection().createQueryRunner``` 를 통해 Transaction 처리를 진행하였습니다.
```const user: User = await this.usersRepository.create(registerUserInfo);``` 이 코드 시점에서 비밀번호가 Beforeinsert 데코레이터를 거쳐 암호화됩니다.
create 와 save 의 차이점은 save 는 DB 에 직접적으로 저장하지만, create 는 객체를 만들기만 합니다.



