---
layout: post
title: "Nest.js 공식문서 - Overview"
subtitle: "공식문서 파먹기"
date: 2021-09-10 12:30:15 +0900
background: '/img/posts/05.jpg'
---

## 들어가기 앞서
Nest.js(이하 Nest)는 타입스크립트와 퓨어 자바스크립트 모두와 호환되지만, 언어의 가장 최신 기술을 사용함에 따라 바닐라 자바스크립트를 사용할때는 바벨 컴파일러가 필요하다.

## Setup
Nest cli를 사용하여 새 프로젝트를 생성하는 것이 가장 간편하다.
``` bash
npm install -g @nestjs/cli
nest new project-name
```

이후 project를 생성하고 나면, node-modules와 몇개의 보일러플레이트 파일들이 설치되고 src/ 디렉토리에 코어파일들이 생성된다

```
src
├─app.controller.spec.ts
├─app.controller.ts
├─app.module.ts
├─app.service.ts
└─main.ts
```

| | |
|-------------------------|---------------------------------------------|
|app.controller.ts        | 하나의 라우트가 있는 기본 컨트롤러             |
|app.controller.spec.ts   | 컨트롤러를 위한 유닛 테스트                   |
|app.module.ts            | 어플리케이션의 루트 모듈                      |
|app.serivce.ts           | 싱글 메소드를 사용하는 기본 서비스             |
|main.ts                  | 어플리케이션의 진입점                         |

이중, main.ts에서는 어플리케이션을 부트스트랩하는 비동기 함수가 포함되어 있다.

``` typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

NestFactory에는 어플리케이션 인스턴스를 생성하는 몇가지 정적 메소드들을 포함하고 있고, create()는 INestApplication 인터페이스를 만족하는 어플리케이션 오브젝트를 반환한다. 
위의 코드를 통해서 들어오는 HTTP요청을 대기하는 HTTP 리스너를 간단하게 생성할 수 있다.

물론 위에서 생성된 프로젝트는 신규 프로젝트를 조금 더 편리하게 만들어줄 뿐인 가설된(scaffolded) 프로젝트라는것을 명심해야 한다.


## Controller
컨트롤러의 목적은 특정 데이터를 수신하는 것이다. 라우팅 메커니즘은 어떤 컨트롤러가 요청을 받는지에 대해서 관리한다. 가끔 개별의 컨트롤러는 하나 이상의 경로를 가지고, 다른 경로는 다른 행동을 취할수 있다.

기본적인 컨트롤러를 생성하기 위해서 클레스와 데코레이터를 사용한다. 데코레이터는 클래스를 필수적인 메타데이터와 연결하고 네스트가 라우팅맵을 작성할수 있도록 한다.

### Routing
이하 예제에서 기본적은 컨트롤러를 정의하는데 필요한 @Contoller() 데코레이터를 사용할 것이다.
@Controller("cats") 데코레이터를 사용함으로서 일련의 경로들을 쉽게 묶을 수 있고, 중복되는 코드들을 최소화 할 수 있다. 

``` typescript
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get('profile')
  findAll(@Req() request: Request): string {
    return 'This action returns cats profile';
  }
}
```

위의 예제에서 GET /cats/profile 요청을 보내면 "This action returns cats profile"이라는 문구를 반환할 것이다. findAll()안에 @Req()를 삽입함으로 요청 객체에 접근할 수 있게 되었다.
요청 객체는 HTTP요청을 나타내고, 쿼리 스트링, 파라메터, HTTP 헤더, 그리고 바디에 대한 속성들을 가지고 있다. 대부분의 경우에서, 이 속성들을 직접 잡을 필요는 없다. 세분화된 데코레이터를 쓸 수 있따.(@Body()나 @Query()).
아래의 표는 제공되는 데코레이터와 기본 플랫폼(Node.js)에 해당되는 객체를 병기해놓았다.

| | |
|-------------------------|---------------------------------------------|
| @Request(), @Req()      | req                                         |
| @Response(), @Res()*    | res                                         |
| @Next()                 | next                                        |
| @Session()              | req.session                                 |
| @Param(key?: string)    | req.params / req.params[key]                |
| @Body(key?: string)     | req.body / req.body[key]                    |
| @Query(key?: string)    | req.query / req.query[key]                  |
| @Headers(name?: string) | req.headers / req.headers[name]             |
| @Ip()                   | req.ip                                      |
| @HostParam()            | req.hosts                                   |

*기본 HTTP 플랫폼(ex. Express / Fastify)에서의 호환성을 위해 Nest는 @Res()와 @Response() 데코레이터를 제공한다. @Res()는 @Response()의 별칭히고, 둘 다 기본 네이티브 플렛폼 resopnse객체 인터페이스를 직접 노출시킨다. 이를 사용하는 경우 기본 라이브러리(ex. types/express)에 대한 타입도 가져와야 모든 어드벤티지를 활용할 수 있다. 메소드 핸들러에 @Res()나 @Response()를 사입읍할 때 해당 핸들러에 대해 Nest를 라이브러리별 모드로 설정하고 응답을 관리해야 한다. 그렇게 하지 않을 경우, Response객체를 만들어서 응답하는(res.json() / res.send()) 경우 HTTP 서버가 멈출 것이다.

### 리소스
이전에 Get 엔드포인트를 생성하였다. 이와 같이, Nest는 모든 HTTP의 표준 메소드들에 대한 데코레이터를 제공한다.
@Get(), @Post(), @Put(), @Delete(), @Patch(), @Options(), @Head().
추가적으로, @All()은 위의 모든 경우에 대한 엔드포인트를 설정한다.


### Status Code
``` typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}

```
앞서 말했다싶이, 응답의 기본 상태 코드는 200이다(Post Request의 응답코드가 201인것을 제외하고). 
응답의 상태 코드를 변경하고 싶은 경우, @HttpCode 데코레이션을 통해서 변경할 수 있다.

### Heaers
헤더를 특정하기 위해서는 @Header()데코레이션을 사용하거나, res.header()를 사용할 수 있다.
```typescript
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

### Redirection
특정 URL로 리다이렉션해주기 위해서 @Rediret()데코레이터를 사용하거나, res.redirect()를 사용하면 된다.
@Redirect()는 두가지 매개변수를 사용하는데, url와 statusCode이고 두가지 모두 선택사항이다. 기본 값의 statusCode는 302(Found)이다.
```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

### Route parameters
라우팅을 할때 URL에서 변동되는 데이터를 받아서 라우팅을 진행해야 할 경우가 있다.
``` typescript
@Get(':id')
findOne(@Param() params): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```
위와 같은 형태로 사용할 수 있따.


### Request Payload
@Body()데코레이터를 사용하기 전에 DTO 스키마를 결정해야 한다. DTO는 데이터가 네트워크를 통해 전송되는 방식을 정의하는 객체로, Typescript 인터페이스를 사용하거나 간단한 클래스를 사용하여 DTO 스키마를 확인할 수 있따. 하지만 여기서는 클래스를 사용하는게 좋은데, 클래스는 ES6표준의 일부이기 때문에 컴파일된 자바스크립트에서 실제 엔티티로 유지되고 인터페이스는 제거되기 때문에 Nest가 런타임 중에 이를 참조할 수 없다. 

``` Typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}

@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}

```


### 실행
컨트롤러 정의 후에 Nest가 CatsController가 존재하는지 알지 못하므로, 결과적으로 이 클래스의 인스턴스를 생성하지 않는다.
컨트롤러는 항상 모듈에 속하므로 @Module()데코레이터 내 controllers배열에 추가함으로서 모듈 클래스에 메타데이터를 첨부하였고, Nest이제 어떤 컨트롤러를 마운트해야하는지 반영할 수 있다.

``` Typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}

```



## 프로바이더

``` Typescript
// cats.service.ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```
``` Typescript
// interfaces/cat.interface.ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```
``` typescript
// cats.controller.ts
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

Nest는 일반적으로 의존성 주입이라고 알려진 강력한 디자인 패턴을 바탕으로 만들어져있다. 이 의존성 주입에 대한 글은 Angular 공식 문서[링크](https://angular.io/guide/dependency-injection)를 읽어보는 것을 추천한다. 


위의 예제에서 만들어진 프로바이더는 컨트롤러와 마찬가지로 app.module.ts에 등록하여야 한다.

``` typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

### Modules
모듈은 @Module()어노테이션이 붙은 클래스로서, @Module()데코레이터는 Nest가 어플레이케이션 구조를 구성하는데 사용되는 메타데이터를 제공한다.

모든 어플리케이션은 최소한 루트 모듈을 포함한 하나 이상의 모듈을 가지고 있으며, 루트 모듈은 Nest가 어플리케이션 그래프를 만드는데 사용되는 시작 포인트이다.

위에서 생성된 컨트롤러와 서비스(프로바이더)와 같은 모듈들을 모으고, 하나의 클래스로 내보낸 후 모든 모듈들을 루트 모듈에서 모은다

``` typescript
// cats/cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```
위에서 만든 컨트롤러와 서비스를 cats.module.ts에서 모은 후, 


``` typescript
// app.module.ts

import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```
루트 모듈에서 모으게 된다

이 경우, 현재의 디렉토리 상황은 다음과 같아진다.
```
src
├─cats
│ ├─dto
│ │  └─create-cat.dto.ts
│ └──interfaces
│    ├─cat.interface.ts
│    ├─cats.controller.ts
│    ├─cats.module.ts
│    └─cats.service.ts
├─app.module.ts
└─main.ts
```

### Shared modules
네스트에서 모듈은 기본적으로 싱글톤이고, 어느 모듈에서 사용하던 항상 같은 인스턴스를 공유할 수 있다.
모든 모듈은 자동적으로 공유 모듈이 되고, 한번 생성한 후 다른 모듈에서 재사용될 수 있다. 
CatsService가 여러 모듈에서 공유되는것을 생각해보자. 이것을 하기 위해서, 먼저 CatsService프로바이더를 아래의 예제 코드처럼 모듈의 exports 배열에 추가해야 한다.

``` typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```
이후 CatsModule를 임포트한 모듈은 CatsService에 접근할 수 있고, 다른 모듈과 같은 인스턴스를 공유하게 된다.

### Dependency Injection
모듈 클래스는 프로바이더에게 의존성 주입을 할 수 있다.
``` typescript 
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```
* 순환 의존성 문제[링크](https://docs.nestjs.com/fundamentals/circular-dependency) 때문에, 모듈 클래스는 그 자신에게 의존성 주입을 할 수 없습니다.

## Middleware
미들웨어는 라우트 핸들러 이전에 호출되는 함수로써, 요청 및 응답 객체에 접근한다. next미들웨어는 일반적으로 next로 명명된다.

Nest 미들웨어는 기본적으로 express의 미들웨어와 동일하다. 
미들웨어는 다음과 같은 일을 할 수 있따.
- 어떠한 코드 실행
- 요청과 응답 객체에 대한 변화
- 요청-응답 사이클의 끝
- 스택에서 next미들웨어 기능 호출
- 만약 미들웨어 기능이 요청-응답 사이클의 끝이 아니라면, next()를 반드시 호출하여야 한다.

미들웨어 또한도 의존성 주입을 사용할 수 있고, 완전히 지원된다. 모듈에서 사용했던 것처럼, constructor를 통해서 진행하면 된다.

### Applying middleware
@Module()데코레이터에는 미들웨어를 위한 자리가 없다. 반면에, 모듈 클레스에서 configure()메소드를 통해 설정할 수 있다. 미들웨어를 포함한 모듈은 NestModule인터페이스를 상속받는다.
LoggerMiddleware을 AppModule레벨에서 설정해보자.

``` typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```
위의 예제는 LoggerMiddleware를 /cat 라우팅 핸들러에 적용하는 것으로서, 이후에 특정 요청 메소드를 설정할 수 있다. 아래의 예제에서는 RequestMethod enum을 불러옴으로서 원하는 요청 메소드 타입을 지정하는데 사용한다.

``` typescript
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

### Middleware consumer
미들웨어 컨슈머는 헬퍼 클레스로써, 여러 내장된 메소드들을 미들웨어를 관리할 수 있게 제공한다. 모든 것들은 fluent style[링크](https://en.wikipedia.org/wiki/Fluent_interface)로 간단하게 연결되어진다. forRoutes() 메소드는 하나의 스트링이나, 복수의 스트링을 가질수 있고, 컨트롤러 클래스나 심지어 복수의 컨트롤러 클래스 또한 가질 수 있다. 많은 경우 여러 컨트롤러들을 콤마로 분리하여 전달해 줄 것이다.

``` typescript
// logger.midleware.ts

import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}

```


``` typescript
// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller.ts';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(CatsController);
  }
}
```


만약 특정 라우팅을 제외하고 싶다면, exclude()메소드를 사용할 수 있다.
``` typescript
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/(.*)',
  )
  .forRoutes(CatsController);
  ```

  물론 위의 로거 미들웨어는 멤버 변수도 없고, 하나의 기능만을 가지고 있다. 이와 같은 경우, 미들에어는 다음과 같이 단축될 수 있다.

  ``` typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```
적용은 위의 의 예제와 동일하게 진행하면 된다.

## Execption filters
Nest에서는 어플리케이션 전체에서 처리되지 않은 모든 예외를 처리하는 예외 레이어가 내장되어 있다.
어플리케이션 코드에서 예외를 처리하지 않으면 이 레이어에서 예외를 포착하여 적절한 응답을 자동으로 보낸다.

기본으로 제공하는 HttpException 클래스를 활용하여 오류 응답을 정의하는 예이다.
``` typescript 
// cats.controller.ts
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```
위의 예제대로 작성시, 응답이 다음과 같이 표시된다.
``` json
{
  "status": 403,
  "error": "This is a custom message"
}

```

대부분의 경우 예외 객체를 커스텀해서 사용할 필요는 없지만, 만들어야 할 경우 다음 예제처럼 Nest에서 기본 제공되는 HttpExecption를 상속받아 고유한 예외 계층을 만드는 것이 좋다.
``` typescript
// forbidden.execption.ts
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```
위에서 생성된 클래스는 HttpException을 상속받아 생성되므로, 이하 메서드들에서 사용할 수 있다.

``` typescript
// cats.controller.ts
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

Nest는 HttpException으로부터 파생된 몇가지의 표준 예외를 제공한다. 이 예외들은 @nestjs/common 패키지에 있으며, 많은 경우의 예외들을 표현한다.
- badRequestException
- UnauthorizedException
- NotFoundException
- ForbiddenException
- NotAcceptableException
- RequestTimeoutException
- ConflictException
- GoneException
- HttpVersionNotSupportedException
- PayloadTooLargeException
- UnsupportedMediaTypeException
- InternalServerErrorException
- NotImplementedException
- ImATeapotException
- MethodNotAllowedException
- BadGatewayException
- ServiceUnavailableException
- GatewayTimeoutException
- PreconditionFailedException


### Exception filters
내장 예외 필터가 자동으로 많은 경우를 처리할 수 있지만, 예외 레이어에 대한 완전한 제어를 원할수도 있다. 예를들어, 로거를 추가하거나 다른 종류의 JSON스키마를 사용하기를 원할 수 있다. Exception filters는 이러한 목적을 위해 디자인 되었으며, 개발자가 원하는 방향으로 클라이언트에게 보내는 응답의 내용을 조절할 수 있다.

``` typescript
///http-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

@Catch(HttpException) 데코레이터는 필요한 메타데이터를 예외 필터에 바인딩하여 특정한 필터가 HttpException 타임의 예외만 찾고 있다는 것을 Nest에게 알린다. @Catch()데코레이터는 단일 매개변수 또는 쉼표로 구분된 목록을 사용할 수 있고, 한번에 여러 타입의 예외에 대한 필터를 설정할 수 있다.

위에서 생성된 필터는 @UseFilters()어노테이션을 통해서 연결할 수 있다.
``` typescript
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```
** 가능한 경우 인스턴스 대신 위의 예제처럼 클래스를 사용하여 필터를 적용하는 것이 좋다.Nest는 전체 모듈에서 동일한 클래스의 인스턴스를 재사용할 수 있으므로 메모리 사용량을 줄이기 때문이다.


## Pipes
파이프는 @Injectable()데코레이터로 어노테이션된 클래스로, 파이프는 PipeTransform 인터페이스를 구현해야 한다.

파이프에는 두가지 선택적 사용법이 있다.
- Transformation: 입력된 데이터를 원하는 형태로 변환한다(string에서 integer)
- Validation: 입력된 데이터가 유효하면 통과시키고, 유효하지 않다면 예외를 발생시킨다.

두가지 경우에서, 파이프는 컨트롤러 라우트 핸들러가 처리하는 매개변수 위에서 작동한다. Nest는 메소드가 실행되기 바로 직전에 끼어들어서 메소드가 처리할 매개변수를 받아 작동하고, 변환이나 검증을 수행한다.

Nest는 개발자가 내부에서 어떻게 동작하는지 몰라도 되는 여러 내장 파이프라인들을 가지고 있으며, 개발자가 커스텀 파이프를 만들수 있도록 지원한다.

내장 파이프 목록
- ValidationPipe
- ParseIntPipe
- ParseFloatPipe
- ParseBoolPipe
- ParseArrayPipe
- ParseUUIDPipe
- ParseEnumPipe
- DefaultValuePipe

### Binding pipes
파이프를 사용하기 위해, 우리는 파이프 클래스의 인스턴스를 적절한 문맥에서 바인딩해야 한다.
예를 들어서, ParseInt파이프를 특정 라우트 핸들러 매소드에 위치시키는 것을 보자.

``` typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```
이 예제는 findOne이라는 매소드를 실행하기 전에, @Param("id")를 통해 가저온 데이터가 Int형으로 바뀔 수 있는지 검사한다. 
만약 Int형으로 변환이 불가능하다면, 다음과 같은 오류를 반환한다.
``` JSON
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

위의 예제에서 우리는 인스턴스가 아니라, ParseIntPipe 클래스를 전달한다. 이렇게 함으로서 인스턴스화의 책임을 프레임워크에게 넘기고, 의존성 주입을 가능하게 한다.


### Custom pipe
가장 간단한 파이프를 만들어 보자. 입력값을 받아서 즉시 같은 값을 반환하는 파이프이다.

``` typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

모든 파이프는 PipeTransform 인터페이스를 충족해야 하기 때문에 transform() 메소드를 구현해야 한다. 이 메소드에는 value와 metatdata 두 개의 배개변수가 있다.

value 매개변수는 현재 처리중인 메소드 인수이고, metadata는 현재 처리중인 메소드 인수의 메타데이터이다.
메타데이터 객체에는 다음과 같은 속성들이 있다.
``` typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

| | |
|----------|--------------------------------------------------------------------------------|
| tpye     | 인수가 @Body인지, @Query인지, @Param()인지 또는 커스텀 매개변수인지 여부를 나타낸다 |
| metaType | 인수의 메타팁을 제공한다.                                                        |
| data     | 데코레이터에 전달된 문자열.                                                      |



### Schema based validation
위에서 만든 유효성 검증 파이프를 조금 더 유용하게 만들어보자. 아래 예제의 create()메소드에서, @Body()가 createCatDto 클래스로 유효한지 검사하게 된다. 
이 단계 이전에, 우리는 들어오는 요청이 유효한 body를 만족하는지 확실히 하고 싶다. 우리는 이 작업을 라우트 핸들러 메소드에서 할 수도 있지만, 이렇게 검증하는것은 단일 책임 원칙(Single Responsibility Rule, SRP)에 위배된다.
다른 접근으로 유효성 검증 클래스를 생성하고 작업을 이 클래스에 위임하는 방법이 될 수 있다. 이것은 매 메소드마다 유효성 검증을 위한 클래스를 사용하는것을 기억하여야 하는 문제점이 있다.

유효성 검증 미들웨어를 만드는 것은 가능할 수 있지만, 전체 어플리케이션에서 동작하는 일반 미들웨어를 생성하는 것은 불가능하다. 미들웨어가 실행 컨텍스트를 인식하지 못하는 문제 때문이다.

그렇다면 어떠한 방법으로 검증 파이프를 개선할 수 있을까


### Object schema validation
깔끔하고, DRY(Do not Reqeat Yourself)하게 객체 유효성 검증을 수행할 수 있는 방법들이 있다. 그 중 일반적인 접근은 스키마에 기반한 검증이다.

Joi 라이브러리는 간단하고 읽기쉬운 API를 활용하여 스키마를 생성할 수 있게 해 준다.
일단 필요한 패키지를 설치하며 시작하자

```
npm install --save joi
npm install --save-dev @types/joi
```

아래의 예제에서는 생성자 매개변수로 스키마를 가지는 간단한 클래스를 생성한다. 그리고 schema.validate()메소드를 적용함으로 값이 스키마와 유효한 구조를 가지는지 검증한다.
``` typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { ObjectSchema } from 'joi';

@Injectable()
export class JoiValidationPipe implements PipeTransform {
  constructor(private schema: ObjectSchema) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const { error } = this.schema.validate(value);
    if (error) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
```

### Binding validation pipes
앞서 ParseIntPipe를 바인딩한 것처럼, 위의 예제 코드를 통해 생성한 유효성 검증 파이프 또한 적용할 수 있다.

``` typescript
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

