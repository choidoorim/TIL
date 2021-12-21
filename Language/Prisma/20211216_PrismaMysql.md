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
$ npm install prisma --save-dev
$ npm i @prisma/client
$ npx prisma
```

아래의 명령어를 통해 초기 설정 Prisma 디렉토리를 생성합니다.
```
$ npx prisma init
```

![prisma snip](https://user-images.githubusercontent.com/63203480/146920282-5b88d356-2c2e-4da0-9add-fb38dc176bb2.png)

## 2. Database 설정

생성된 ```.env``` 파일에서 ```DATABASE_URL``` 을 Mysql 관련 설정으로 변경해줍니다.

```
DATABASE_URL="mysql://USER:PASSWORD@HOST:PORT/DATABASE_NAME"
```

## 3. Prisma Schema 
