# Prisma - 중복되는 로우 쿼리 재사용하기
Prisma 에서 Raw Query 를 사용할 때 공통되는 부분이 생기기 마련입니다.   
예를 들어 아래와 같은 2개의 쿼리를 사용하고 있다고 가정을 해봅시다.

```typescript
this.prismaService.$queryRaw`select * from User where id = ${userId};`;
this.prismaService.$queryRaw`select * from User where age < ${maxAge};`;
```

```select * from User``` 쿼리가 현재 공통으로 사용되고 있습니다.
만약 ```select * from User``` 을 문자열로 만들어서 추가해주면 어떻게 될까요?

```typescript
const selectQuery: string = 'select * from User';

this.prismaService.$queryRaw`${selectQuery} where id = ${userId};`;
this.prismaService.$queryRaw`${selectQuery} where age < ${maxAge};`;
```

아래와 같은 syntax error 가 발생합니다. [prisma issue](https://github.com/prisma/prisma/issues/5083)

```text
"db error: ERROR: syntax error at or near \"$1\" // error message
```

분명히 공통되는 string type 을 넘겨서 사용했는데 왜 작동하지 않을까요?      
[공식문서](https://www.prisma.io/docs/concepts/components/prisma-client/raw-database-access#tagged-template-helpers) 에서 확인해보면 ```$queryRaw``` 메서드는 ```Prisma.sql``` Helper 나 템플릿 문자열만 허용한다고 합니다.

```text
in fact, the $queryRaw method will only accept a template string or the Prisma.sql helper.
```

## 해결방법
**Prisma.sql** Helper 를 사용해서 공통되는 Query 를 재사용할 수 있게 변경해보도록 하겠습니다.

```typescript
import { Prisma } from '@prisma/client';
//...

const selectSql: Prisma.Sql = Prisma.sql`select * from User`;

this.prismaService.$queryRaw`${selectSql} where id = ${userId};`;
this.prismaService.$queryRaw`${selectSql} where age < ${maxAge};`;
```

위와 같이 변경한다면 문제없이 queryRaw 메서드가 동작하는 것을 확인하실 수 있을 것입니다.

만약 Query 를 return 하는 메서드를 만들 경우에는 아래와 같이 할 수 있습니다.

```typescript
function userSelectWhereSql(userId: number): Prisma.Sql {
  return Prisma.sql`select * from User where id = ${userId};`; 
}
```