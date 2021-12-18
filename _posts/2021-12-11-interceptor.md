---
layout: post
title: 'Nest.js의 인터셉터'
subtitle: ''
date: 2021-12-11 12:30:15 +0900
background: '/img/posts/05.jpg'
---

## Interceptor?

인터셉터는 java spring에서 AOP, aspect oriented programming 즉 관점 지향 프로그램에서 영향을 받은 기술이다.
즉, 서비스 로직에서 핵신점은 관점은 유지한 채로, 부가적인 관점들은 모듈화해서 재사용한다는 말이다.
예를 들어, 데이터베이스 연결, 로깅 파일 입출력 등이 부가적인 관점들이 될 수 있겠다.

이러한 부가적인 관점들이 기존의 서비스 로직에 산재되어 있고, 중복해서 사용되는 것들을 흩어진 관심사(Crosscutting concerns)라고 한다.
AOP는 이러한 흩어진 관심사를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는것이 취지이다.

당장 사용하지 않을 spring의 기술에 대해서는 접어보고, Nest.js에서 이 관심사를 분리하고 어떻게 재사용하는지 보도록 하자.

### Interceptor의 역할

Nestjs에서 interceptor는 메소드 실행 전, 후 추가 로직 바인딩, 함수에서 반환된 결과를 반환, 함수에서 던져진 예외 반환, 기본 기능 동작 확장, 특정 조건에 따라 기능을 완전히 재정의 하는 기능을 수행할 수 있다.

우선 인터셉터의 기본 개념을 예제를 통해서 보도록 하자.

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common'
import { Observable } from 'rxjs'
import { tap } from 'rxjs/operators'

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...')

    const now = Date.now()
    return next
      .handle()
      .pipe(tap(() => console.log(`After... ${Date.now() - now}ms`)))
  }
}
```

해당 인터셉터를 컨트롤러에 등록하려면

```typescript
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

와 같은 식으로 등록할 수 있다.

위와 같은 구성으로 정의된 컨트롤러에서, 각 라우트 핸들러는 LoggingInterceptor를 사용하게 된다.

예를 들어, GET /cats를 호출하게 되면 다음과 같은 출력이 생성된다.

```
Before...
After 1ms
```

위의 코드에서 가장 중요하다고 생각되는 부분은 Observable 객체의 관리라고 생각된다.
처음 듣는 생소한 개념인데, 일단 Rxjs에서 파상된 observable 객체에 대해서, rxjs에서도 한번 알아보자.
그래야 위의 코드에 대해서 알 수 있을 것 같다.

```typescript
import { of } from 'rxjs'
import { tap, map } from 'rxjs/operators'

const source = of(1, 2, 3, 4, 5)

const example = source
  .pipe(
    map((val) => val + 10),
    tap((val) => console.log('tap value ' + val)),
  )

  .subscribe((val) => console.log(val))
// output
// tap value 11
// 11
// tap value 12
// 12
// tap value 13
// 13
// tap value 14
// 14
// tap value 15
// 15
```

위의 예제에서, source를 stream으로 정의한다.
이후 pipe 함수를 통해서 스트림을 읽고, 이 데이터를 map과 tap을 통해서 컨트롤하게 된다.
map을 통해서 읽어온 데이터에 10을 더해주는 가공을 처리하고, tap을 사용해 스트림에 영향을 주지 않는 작업을 처리한다(로깅 등)
이제 이렇게 만든 스트림은 subscribe으로 구독한 observer에서 읽게 되고, 그 값을 console에서 뿌려주는 작업을 처리하게 된다.

슬적슬적 그려보면 이런 느낌이 아닐까..?

![rxjs](/img/posts/21_12_11/rxjs.png)

아무튼 rxjs가 어떻게 처리되는지는 알았다. 그럼 Interceptor에서 어떻게 처리될지도 한번 생각해보자.

1. controller로 요청이 들어온다.
2. Interceptor가 요청을 먼저 받는다
3. Interceptor 내의 context: ExecutionContext 를 가지고 들어오는 요청에 대한 데이터를 가공한다.
4. service 로직쪽으로 데이터를 보내주고, 비즈니스 로직에서 데이터를 처리한다.
5. 비즈니스 로직에서 처리가 완료된 데이터는 클라이언트로 가기 전, Interceptor에서 next.handle():observable을 통해서 다시 한번 가공된다.
6. 클라이언트에 데이터가 도착한다.

이와 같은 방법을 통해서 데이터가 처리되기 전, 처리된 후 가공할 수 있음을 파악했다.

그럼 이와 같은 방법을 사용하는 라이브러리를 생각해보자

## Interceptor 활용한 라이브러리

현재 진행중인 프로젝트에서 파일 업로드(이미지 등)을 지원해야 하는 문제가 있다.
이럴때 사용하는 multer 라이브러리는 Interceptor로 랩핑되어 제공하고 있는데, 해당 현 프로젝트에서 사용되는 예시를 보자

```typescript
// post.controller.ts
@Post()
	@UseInterceptors(
		FilesInterceptor("images", 10, {
			storage: diskStorage({
				destination: "./public/images",
				filename: (req, file, cb) => {
					const randomName = Array(32)
						.fill(null)
						.map(() => Math.round(Math.random() * 16).toString(16))
						.join("");
					cb(null, `${randomName}${extname(file.originalname)}`);
				},
			}),
		}),
	)
	async createPost(
		@UploadedFiles() files: Array<Express.Multer.File>,
		@Body() post: { content: string },
		@Req() req,
	): Promise<ExecutionResult> {
		console.debug(files);
		console.debug(post);
        ...
    }


```

FilesInterceptor라는 인터셉터는 multi-part로 들어온 파일들을 이름을 바꾼 후 로컬에 저장하고, 파일들에 대한 정보를 request.files 혹은 nest에서는 데코레이터 @UploadFiles를 통해 생성된 files라는 객체에 저장된다.
form data에 저장되어있던 다른 값들은 req.body에 저장되고, 이러한 값들은 파일 업로드 후 데이터베이스 저장 등에 유용한 값으로 사용할 수 있다.

## 결론

Interceptor를 잘 사용하면 여러가지 경우에서 유용하게 쓸 수 있을것 같다.
비록 지금 생각나는건 파일 데이터 처리, 혹은 로깅 등 밖에 안되지만....

그래도 미리 배워놓으면 나중에 쓸 수 있는 칼이 하나 더 생긴다고 생각해야겠다

rxjs에 대해서 새롭게 알게 된 것도 좋았다!!
