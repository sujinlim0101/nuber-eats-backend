## Nestjs

scalable한 노드의 서버사이드 애플리케이션을 위한 프레임워크이다. 프로그레시브 JavaScript를 사용하고 TypeScript로 구축되고 완벽하게 지원하며(그러나 여전히 개발자가 순수 JavaScript로 코딩할 수 있음) OOP(객체 지향 프로그래밍), FP(기능 프로그래밍) 및 FRP(기능 반응 프로그래밍)의 요소를 결합합니다.

내부적으로 Nest는 Express(기본값)와 같은 강력한 HTTP 서버 프레임워크를 사용하며 선택적으로 Fastify도 사용하도록 구성할 수 있습니다.

Nest는 개발자와 팀이 테스트 가능성이 높고 확장 가능하며 느슨하게 결합되고 유지 관리가 쉬운 애플리케이션을 만들 수 있도록 하는 즉시 사용 가능한 애플리케이션 아키텍처를 제공합니다. 아키텍처는 Angular에서 크게 영감을 받았습니다.

- **installation**

```jsx
$ npm i -g @nestjs/cli
$ nest new project-name
```

📄 main.ts

```jsx
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

nest application의 인스턴스를 만들기 위해서, 우리는 NestFactory를 씁니다. 이건 몇몇 static methods를 제공하는데, 애플리케이션의 인스턴스를 만드는것을 돕습니다.

create() 메서드는 INestApplication 인터페이스를 충족하는 애플리케이션 객체를 반환합니다. 이 개체는 다음 장에서 설명하는 메서드 집합을 제공합니다.

## ****Controllers****

![Controllers_1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2ccfdea0-3857-4b93-bc52-9db288beb941/Controllers_1.png)

컨트롤러의 목적은 애플리케이션에 대한 특정 요청을 수신하는 것입니다. 라우팅 메커니즘은 어떤 컨트롤러가 어떤 요청을 수신하는지 제어합니다. 종종 각 컨트롤러에는 둘 이상의 경로가 있으며 다른 경로는 다른 작업을 수행할 수 있습니다.

- ****Routing****

다음 예제에서는 기본 컨트롤러를 정의하는 데 필요한 @Controller() 데코레이터를 사용할 것입니다.

cats.controller.ts

```
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

- ****Request object****

핸들러는 종종 클라이언트 요청 세부 정보에 액세스해야 합니다. Nest는 기본 플랫폼(기본적으로 Express)의 요청 개체에 대한 액세스를 제공합니다. 핸들러의 서명에 @Req() 데코레이터를 추가하여 요청 개체를 삽입하도록 Nest에 지시하여 요청 개체에 액세스할 수 있습니다.

cats.controller.ts

**JS**

```
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}

```

- ****Resources****

Nest는 @Get(), @Post(), @Put(), @Delete(), @Patch(), @Options() 및 @Head()와 같은 모든 표준 HTTP 메서드에 대한 데코레이터를 제공합니다. 또한 @All()은 모든 것을 처리하는 끝점을 정의합니다.

## ****Providers****

provider의 주요 아이디어는 종속성으로 주입 될수 있다는 것입니다. 즉 객체는 서로 다양한 관계를 생성할 수 있으며 객체의 인스턴스를 연결하는 기능은 대부분 nest 런타임 시스템에 위임될 수 있습니다.

- Service

간단한 `CatsService`. 이 서비스는 데이터 저장 및 검색을 담당하며 에서 사용하도록 설계되었으므로 `CatsController`공급자로 정의하기에 좋은 후보입니다.

cats.service.ts

**JS**

```
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];create(cat: Cat) {
    this.cats.push(cat);
  }findAll(): Cat[] {
    return this.cats;
  }
}
```

CLI를 사용하여 서비스를 생성하려면

```
$ nest g service cats
```

명령을 실행하기만 하면 됩니다.

interfaces/cat.interface.ts

```
export interface Cat {
  name: string;
  age: number;
  breed: string;
}

```

cats.controller.ts**JS**

```
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

CatsService는 클래스 생성자를 통해 주입됩니다. private 구문의 사용에 주목하십시오. 이 shorthand을 사용하면 동일한 위치에서 cat service 멤버를 즉시 선언하고 초기화할 수 있습니다.

- ****Dependency injection****

Nest는 일반적으로 종속성 주입으로 알려진 강력한 디자인 패턴을 기반으로 구축되었습니다. 공식 [Angular](https://angular.io/guide/dependency-injection) 문서에서 이 개념에 대한 훌륭한 기사를 읽는 것이 좋습니다.

- ****Provider registration****

우리가 CatsService를 정의해주었다. 그리고 우리는 CatsController라는 consumer가 있다. 우리는 이 서비스를 Nest 에 등록하고, inject 해주자.

app.module.ts

**JS**

```
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

## ****Modules****

모듈은 @Module() 데코레이터로 주석이 달린 클래스입니다. @Module() 데코레이터는 Nest가 애플리케이션 구조를 구성하는 데 사용하는 메타데이터를 제공합니다.

각 응용 프로그램에는 루트 모듈인 하나 이상의 모듈이 있습니다. 루트 모듈은 Nest가 애플리케이션 그래프를 빌드하는 데 사용하는 시작점입니다. Nest가 모듈과 공급자 관계 및 종속성을 해결하는 데 사용하는 내부 데이터 구조입니다. 매우 작은 응용 프로그램에는 이론적으로 루트 모듈만 있을 수 있지만 일반적인 경우는 아닙니다. 구성 요소를 구성하는 효과적인 방법으로 모듈을 사용하는 것이 좋습니다. 따라서 대부분의 응용 프로그램에서 결과 아키텍처는 각각 밀접하게 관련된 기능 집합을 캡슐화하는 여러 모듈을 사용합니다.

## 환경변수

dotenv는 .env 파일에서 환경변수를 로드한다. 하지만 Nest에서는 `@nestjs/config` 를 설치하여 관리한다.

ConfigModule을 import하고 app moule에서 사용한다.

cross-env는 macos인지, 윈도우인지 상관없이 환경변수를 쓸수 있게 해준다.

## Validating configservice

**JavaScript용 가장 강력한 스키마 설명 언어 및 데이터 유효성 검사기.**

[https://www.npmjs.com/package/joi](https://www.npmjs.com/package/joi)

## Module

모듈은 @Module 데코레이터로 정의된 클래스이며, 이 모듈들의 조합을 통해 NestJS의 구조가 결정된다.

애플리케이션은 반드시 하나의 Root Module이 있어야 작동하며 그 외의 기능들을 구현하는 모듈들을 Root Module로 연결하여 사용한다. @Module 데코레이터는 아래의 인자들을 받아 모듈을 구성할 수 있다.

- `providers`: Injectable된 클래스들을 인스턴스화 하고 인스턴스는 모듈 안에서 최소한으로 공유된다.
- `controllers`: 해당 모듈에서 사용되는 컨트롤러의 모음을 인스턴스화 해준다.
- `imports`: 다른 모듈들을 임포트하여 연결해준다. export된 provider가 있다면 사용할 수 있다.
- `exprots`: 임포트하는 모듈에서 사용할 provider를 지정할 수 있다.

 

## Static Module, Dynamic Module

forRoot가 없고 configuration을 따로 하지 않는 모듈이다. dynamic module은 configuartion을 하여 리턴된다.


# 데이터베이스

# **ORM**

ORM (Object-relational mapping)은 객체지향 프로그래밍(Object-Oriented-Programming) 과 관계형 데이터베이스(Relational-Database)사이ㄴ의 호환되지 않는 데이터를 변환하는 시스템이다. 객체지향 프로그래밍은 Class를 사용하고 관계형 데이터베이스는 Table을 사용한다.

# **TypeOrm**

TypeOrm 이란 TypeScript 와 JavaScript(ES5 , ES6 , ES7) 용 ORM이다. MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, WebSQL 데이터베이스를 지원한다. 여기서는 PostgreSQL을 사용할 것이다.

## **PostgreSQL이란?**

PostgreSQL은 오픈 소스 객체-관계형 데이터베이스 시스템(ORDBMS)으로, Enterprise급 DBMS의 기능과 차세대 DBMS에서나 볼 수 있을 법한 기능들을 제공한다.약 20여년의 오랜 역사를 갖는 PostgreSQL은 다른 관계형 데이터베이스 시스템과 달리 연산자, 복합 자료형, 집계 함수, 자료형 변환자, 확장 기능 등 다양한 데이터베이스 객체를 사용자가 임의로 만들 수 있는 기능을 제공함으로써 마치 새로운 하나의 프로그래밍 언어처럼 무한한 기능을 손쉽게 구현할 수 있다.

Postico는 데이터 베이스 내용을 시각화해준다.

## MacOS Setup

**PostgreSQL,** Postico 설치하고, Postico에서 데이터 베이스 추가.

**PostgreSQL이 런되고 있는 상태여야한다. PostgreSQL에 해당 데이터 베이스 더블 클릭하면 터미널이 뜨고 `ALTER USER 'sujinlim' WITH PASSWORD "12345"`  해줌**

## TypeORM Setup

nestjs와 커넥트 할 계획.


# TypeORM and Nest

Entity는 클래스인데, 데이터베이스 테이블에 맵핑되는 것이다.

nestjs 엔티티와 graphql 엔티티와 비슷하다.

user.entity.ts

```jsx
import { Entity, Column, PrimaryGeneratedColumn, OneToMany } from 'typeorm';
import { Photo } from '../photos/photo.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @OneToMany(type => Photo, photo => photo.user)
  photos: Photo[];
}
```

graphql entity

```jsx
@ObjectType()
export class Restaurant {
  @Field(type => String)
  name: string;

  @Field(type => Boolean)
  isVegan: boolean;

  @Field(type => String)
  address: string;

  @Field(type => String)
  ownersName: string;
}
```

둘을 combine할수 있음.

```jsx
@ObjectType()
@Entity()
export class Restaurant {
  @Field((type) => String)
  @Column()
  name: string;

  @Field((type) => Boolean)
  @Column()
  isVegan: boolean;

  @Field((type) => String)
  @Column()
  address: string;

  @Field((type) => String)
  @Column()
  ownersName: string;
}
```

TypeORM에 우리가 만든 Entity가 어디있는지 알려줘야한다. 그렇게 하기 위해서 두가지 방법이 있다.

1. TypeOrmModule.forRoot 쪽에 entities 필드를 추가하고 Resatuarnt 엔티티를 추가한다.

## Repository

### ****Data Mapper vs Active Record****

TypeOrm에서 사용 되는 일종의 패턴을 의미한다.

## 1. Active Record Pattern

Active Record 패턴은 모델 안에 모든 쿼리들을 정의해두고 CRUD 작업들을 모델의 메소드를 통해 실행하게 된다.

즉, Active Record 패턴은 모델 안에서 데이터베이스에 접근하는 패턴이라고 볼 수 있을 거 같다.

```jsx
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User extends BaseEntity {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

    static findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }
}
```

## Data Mapper Pattern

Data Mapper 패턴을 사용하면, 모든 쿼리들을 Repository 라는 분리된 클래스로 관리하게 되고 이를 통해 CRUD작업을 하게 된다. 즉, Data Mapper패턴에서 우리의 모델들은 그저 프로퍼티를 정의하는 "Dumb" 한 상태가 된다.

```jsx
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```

```jsx
import {User} from "../entity/User";

const userRepository = connection.getRepository(User);

// example how to save DM entity
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await userRepository.save(user);

// example how to remove DM entity
await userRepository.remove(user);

// example how to load DM entities
const users = await userRepository.find({ skip: 2, take: 5 });
const newUsers = await userRepository.find({ isActive: true });
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

레포지토리는 엔티티와 상호작용하는걸 담당한다.

User 엔터티 사용을 시작하려면 forRoot() 메서드 옵션 모듈의 엔터티 배열에 이를 삽입하여 TypeORM에 이에 대해 알려야 합니다.

이제 모듈을 보죠.  `UsersModule`:

```jsx
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

이 모듈은 forFeature() 메서드를 사용하여 현재 범위에 등록된 저장소를 정의합니다. 그 자리에서 @InjectRepository() 데코레이터를 사용하여 UsersRepository를 UsersService에 주입할 수 있습니다.

```jsx
// users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: string): Promise<User> {
    return this.usersRepository.findOne(id);
  }

  async remove(id: string): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```

## Recap

TypeOrm에서 엔티티로 Restaurant을 가지고 있다. 이게 데이터베이스가 된다.

resaturnat 모듈에서 imports 부분에 TypeOrmModule의 forFeature로 Reastaurant이다. 

resolver에서 constructor로 레스토랑 서비스를 가져왔다. 이 서비스는 레스토랑 모듈에서 provider로 제공된다. 그리고 resolver에서 inject된다.

서비스로 가면 restraunt 레포지토리를 써서 db에서 원하는 값을 가져온다.

## ****Mapped types****

CRUD(Create / Read / Update / Delete)와 같은 기능을 구축할 때 기본 엔터티 유형에 대한 변형을 구성하는 것이 종종 유용합니다. Nest는 이 작업을 보다 편리하게 하기 위해 유형 변환을 수행하는 여러 유틸리티 함수를 제공합니다.

—> partial / Pick / Omit / Intersection