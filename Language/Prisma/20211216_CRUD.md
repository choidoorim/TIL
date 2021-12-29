# Prisma, Mysql 시작하기
# Prisma 란?
[공식 홈페이지](https://www.prisma.io/docs/concepts/overview/what-is-prisma#what-is-prisma) 에서 확인해보면 차세대 오픈 소스 ORM 으로 아래와 같은 구성으로 이뤄져있다고 합니다.

- Prisma Client : NodeJS 와 TypeScript 전용 Type Safe 및 자동 생성 쿼리빌더
- Prisma Migrate : Migration system, 데이터 모델링
- Prisma Studio : GUI 를 통해 DB 를 수정할 수 있는 기능

Prisma Client 는 모든 NodeJS 또는 Typescript 백엔드 어플리케이션에서 사용이 가능하다고 합니다.
그리고 PostgreSQL, MySQL, MariaDB, SQLite 등의 데이터베이스를 지원합니다.

## 왜 Prisma 를 사용할까?
Prisma 의 주요 목적은 데이터베이스 작업 시 개발자의 생산성을 높이는 것이라고 합니다.
아래는 이러한 목적을 달성하기 위한 몇가지 이유들입니다. 자세한 내용은 [공식문서](https://www.prisma.io/docs/concepts/overview/why-prisma/) 에 잘 설명되어 있습니다.

- 관계형 데이터를 매핑하는 것 대신 객체를 사용
- 복잡한 모델 객체를 피하기 위해 클래스가 아닌 쿼리를 사용
- 데이터베이스 및 어플리케이션 모델을 위한 Single source of Truth 이론(정보의 중복, 비적합성 등의 문제를 해결하기 위한 이론)
- 흔한 함정과 안티패턴을 막기 위한 단단한 제약조건
- 올바른 것을 쉽게 만드는 추상화
- 컴파일 시 유효성 검사를 할 수 있는 Type Safe 데이터베이스 쿼리
- 단순 노동을 줄일 수 있도록 하여(Boilerplate Code) 개발자들이 어플리케이션에 집중할 수 있습니다
- 공식문서를 찾아볼 필요 없이 코드 Editor 에서 자동 완성 기능

## 1. Prisma 설치하기
prisma 를 설치하고, 로컬에서 호출합니다.
```
npm install prisma --save-dev
npm i @prisma/client
npx prisma
```

아래의 명령어를 통해 초기 설정 Prisma 디렉토리를 생성합니다.
```
npx prisma init
```

![prisma snip](https://user-images.githubusercontent.com/63203480/146920282-5b88d356-2c2e-4da0-9add-fb38dc176bb2.png)

## 2. Database 설정

생성된 ```.env``` 파일에서 ```DATABASE_URL``` 을 Mysql 관련 설정으로 변경해줍니다.

```
DATABASE_URL="mysql://USER:PASSWORD@HOST:PORT/DATABASE_NAME"
```

## 3. Prisma Schema 
```schema.prisma``` 파일은 Prisma 설정에 대한 메인 파일입니다. 크게 'Generator', 'Datasource', 'Data Model' 로 나눌 수 있습니다.

- [Generators](https://www.prisma.io/docs/concepts/components/prisma-schema/generators) : Prisma Client 를 기반으로 생성되어야 하는 클라이언트를 정의
- [Datasource](https://www.prisma.io/docs/concepts/components/prisma-schema/data-sources) : Prisma 가 연결해야 할 DB 에 대한 정보를 정의
- [Data Model](https://www.prisma.io/docs/concepts/components/prisma-schema/data-model) :  데이터 모델(테이블)을 정의

#### prisma/schema.prisma
```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String?
  role    Role     @default(USER)
  posts   Post[]
  profile Profile?
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String
  user   User   @relation(fields: [userId], references: [id])
  userId Int
}
```

Relation 을 설정하는 것은 [공식문서](https://www.prisma.io/docs/concepts/components/prisma-schema/relations) 에 자세히 설명되어 있습니다.

## 4. Prisma Migrate 
명령어를 통해 Prisma Schema 에서 정의한 설정과 모델을 바탕으로 **'Migrate'** 를 할 수 있습니다.

- Migrate 란? : 추가, 변경, 삭제 등을 진행한 Model 을 데이터베이스에 전송하여 반영하는 것

#### 명령어
```
npx prisma migrate dev --name HISTORY-NAME
```
HISTORY-NAME 에 Migration 시 남길 History 명을 입력해주면 됩니다.

명령을 수행하면 Model 에 대한 내용이 DB 에 Migration 되고, ```prisma/migrations``` 경로에 히스토리가 남는 것을 확인할 수 있습니다.

![DB-migrate-result](https://user-images.githubusercontent.com/63203480/147036928-302cde87-235b-4781-adf7-cc91c80ae6dd.png)

![image](https://user-images.githubusercontent.com/63203480/147037110-446a712a-67e9-4cbc-bf00-3b1ccb6436b0.png)

데이터베이스와 모델을 Migration 을 수행하기 전에 수정이 필요한 상황이 있을 수 있습니다.
예를 들면,

- 중요한 리팩토링
- 필드의 이름을 변경
- 관계의 방향을 수정
- Prisma Schema 언어로 나타낼 수 없는 기능을 추가하고자 할 때 

이러한 상황에는 ```--create-only``` 라는 명령어를 추가해주면 됩니다.
```
npx prisma migrate dev --create-only
```
옵션을 추가하여 migration 을 진행하면, 변경 내용이 **DB 에 즉시 반영되지 않고 히스토리만 남습니다**.     
최종적으로 수정된 Schema 로 Migration 할 때는 ```npx prisma migrate dev``` 을 실행하면 됩니다.

## 5. Prisma Client 로 DataBase 조작
Prisma 와 DB 를 연결하고, Model 을 적용시키는 것 까지는 했습니다.   
DB 를 제어, 조작하기 위해서는 [Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client) 가 필요합니다.
만약 Prisma Client 가 설치되어 있지 않다면 설치해줘야 합니다.
```
npm i @prisma/client
```
Prisma Client 설치가 끝났다면, 아래의 명령어를 통해 Prisma Client 를 생성해줘야 합니다.
```
npx prisma generate
```
명령어가 수행되면, Prisma 는 Model 에 대한 스키마를 읽고, Prisma Client 에 반영합니다.

![prisma-generate](https://user-images.githubusercontent.com/63203480/147351509-4205d3af-b5f9-40d9-ae78-590f721222e7.png)

반드시 기억해야 할 것이 있는데, Prisma Client 의 실행흐름은 위의 그림과 같습니다. 
따라서 **Prisma Schema 가 변경된다면 generate 명령어를 통해 Prisma Client 를 업데이트** 해줘야 합니다. 

## Prisma Client 를 사용하여 CRUD 수행
[NestJS 공식문서](https://docs.nestjs.kr/recipes/prisma) 를 참고해보면, PrismaClient 를 인스턴스화 하고, 데이터베이스에 연결하는 새로운 PrismaService 를 만드는 방법을 소개합니다.
[Prisma CRUD 공식문서](https://www.prisma.io/docs/concepts/components/prisma-client/crud) 를 보면 더 자세한 CRUD 방법을 소개하고 있습니다.

#### prisma.service.ts
```typescript
import { INestApplication, Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient
  implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: INestApplication) {
    this.$on('beforeExit', async () => {
      await app.close();
    });
  }
}
```

저는 prisma.service.ts 를 기능 별 데이터베이스 쿼리를 수행하도록 하였습니다.

### Read Data
```findUnique()``` 메서드를 사용하여 id 가 userIdx 인 데이터를 찾을 수 있습니다.

#### prisma.service.ts
```typescript
//...
export class PrismaService extends PrismaClient
    //...
    async findUserById(userIdx: number) {
        return await this.user.findUnique({
            where: {
                id: userIdx,
            }
        })
    }
}
```

#### user.service.ts
```typescript
@Injectable()
export class UserService {
    constructor(private prisma: PrismaService) {}

    async findUserById(userIdx: number) {
        return this.prisma.findUserById(userIdx);
    }
}
```

### Create Data
```create()```, ```createMany()``` 메서드를 사용해서 데이터를 생성할 수 있습니다.

#### prisma.service.ts
```typescript
//...
export class PrismaService extends PrismaClient
    //...
    async createUser(createUserReq: Prisma.UserCreateInput): Promise<User|null> {
        return await this.user.create({
            data: {
                email: createUserReq.email,
                name: createUserReq.name
            }
        })
    }
}
```
```
// result
{
    "id": 3,
    "email": "qwe@qwe.com",
    "name": "qwe123123"
}
```

```createMany()``` 메서드를 사용하면 여러 개의 데이터를 한 번에 생성할 수 있습니다.
```typescript
//...
async createUser(){
    return await this.user.createMany({
        data: [
            { email: 'qwe@qwe.com', name: 'qwe' },
            { email: 'asd@asd.com', name: 'asd' },
            { email: 'ttt@ttt.com', name: 'ttt' },
        ]
    })
}
```

```
// result
{
  count: 3
}
```

####  user.service.ts
```typescript
//...
async createUser() {
    return await this.prisma.createUser();
}
```

### Update Data
```update()```, ```updateMany()``` 메서드를 사용하여 데이터를 업데이트 할 수 있습니다.     
```updateMany()``` 도 ```createMany()``` 와 동일하게 return 값이 Count 로만 이뤄져 있습니다.

#### prisma.service.ts
```typescript
//...
export class PrismaService extends PrismaClient
    //...
    async updateUser(): Promise<User> {
        return await this.user.update({
            where: {
                id: 1,
            },
            data: {
                email: 'qwe@qwe.com'
                name: 'qweqwe123',
            },
        });
    }
}
```

####  user.service.ts
```typescript
//...
async updateUser() {
    return await this.prisma.updateUser();
}
```

### DELETE Data
```delete()```, ```deleteMany()``` 메서드를 사용하여 데이터를 삭제할 수 있습니다.    
deleteMany 도 CreateMany 와 동일하게 return 값이 Count 로만 이뤄져 있습니다.


#### prisma.service.ts
```typescript
//...
async deleteUser(userIdx: number): Promise<User> {
    return await this.user.delete({
        where: {
            id: userIdx
        }
    });
}
```

#### user.service.ts
```typescript
//...
async deleteUser(userIdx) {
    return await this.prisma.deleteUser(userIdx);
}
```