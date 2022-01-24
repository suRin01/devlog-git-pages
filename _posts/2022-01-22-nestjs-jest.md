---
layout: post
title: 'Jest에서 프로바이더 의존성 주입으로 DB연결되는 문제'
subtitle: ''
date: 2022-01-16 12:30:15 +0900
background: '/img/posts/05.jpg'
---


## 문제 원인

Nest 환경에서 Jest를 사용해 unit test를 진행하려고 하였다.
그런데 테스트용 모듈을 생성해주는 과정에서, 기존 필요로 하는 모든 프로바이더들과 모듈을 import하면 자동으로 데이터베이스 커넥션 풀이 생성되고, 
실제 디비와 통신하게 되어 unit test의 고립성이 깨지게 되고 결국 실제 데이터에 접근하여 조작되고, 또 테스트를 시행하는데 더 많은 시간이 필요하게 된다.

이러한 문제점을 방지하기 위해 필요로 하는 모듈들을 어떻게 mocking하고 사용하는지에 대해서 몰랐고 unit test의 고립성에 대해 필요성을 몰랐기 때문에
mapper 프로바이더 등 데이터베이스나 타 어플리케이션과 통신하는 프로바이더들을 mocking하지 않고 그대로 테스트를 시행했었다.

나의 경우에서는 테스트를 진행할수록 db에 필요없는 데이터가 쌓이기 시작했고, 이 데이터들을 clean up 해주는데 또 시간을 쓰다보니 개발 이외의 잡다한 일에 더 시간을 뻇기게 되었다.

그래서 unit test에 대해서 다시 한번 읽어보다가, 고립성에 대해서 말하고 있는 포스트를 보니 내 이야기와 딱 알맞는 이야기여서 프로바이더들을 mocking하는 방법을 찾아보려고 하였다.


### 문제 해결 전

``` typescript
import { UserController } from "../../controller/user.controller"
import { UserServcie } from "../../service/user.service"
import { Test } from '@nestjs/testing';
import { Mapper } from '../../mapper/mapper';
import { newUser, user, userCreationResult } from "../testData"

describe('UserController', () => {
    let userController: UserController;
    let userService: UserServcie;

    beforeEach(async () => {
      const moduleRef = await Test.createTestingModule({
          controllers: [UserController],
          providers: [UserServcie, Mapper],
        }).compile();

        userService = moduleRef.get<UserServcie>(UserServcie);
        userController = moduleRef.get<UserController>(UserController);
    });

    describe('findOne', () => {
        it('should return one user array', async () => {
            jest.spyOn(userService, 'getUser').mockResolvedValue(user);
        
            expect(await userController.getUser("test")).toEqual(user);
        });
    });

    describe("createOne", ()=>{
        it("make one user", async ()=>{
            jest.spyOn(userService, "createUser").mockResolvedValue(userCreationResult);

            expect(await userController.createUser(newUser)).toEqual(userCreationResult);
        })
    })
});


```

테스트를 시행하게 되면, 일단 데이터베이스 커넥션 풀을 생성하고, 풀에서 커넥션을 생성한 후 가져오고, 그 모든 일련의 과정이 끝난 후에 드디어 테스트가 시작된다.

이 경우에서 테스트를 시행하는데 케이스당 평균적으로 18초 정도가 소요되었으며, 
2개의 테스트가 병렬로 시행되었다는 것을 생각하면 테스트 케이스가 많아질 경우 얼마나 많은 부하와 시간이 소요될지 가늠할 수 없을 정도이다.

이랬던 테스트를, 이렇게 바꿔보자



### 문제 해결 후
``` typescript

import { UserController } from "../../controller/user.controller"
import { UserServcie } from "../../service/user.service"
import { Test } from '@nestjs/testing';
import { Mapper } from '../../mapper/mapper';
import { newUser, user, userCreationResult } from "../testData"

const MockMapperRepository = ()=>({
    mapper: jest.fn()
})

describe('UserController', () => {
    let userController: UserController;
    let userService: UserServcie;

    beforeEach(async () => {
      const moduleRef = await Test.createTestingModule({
          controllers: [UserController],
          providers: [UserServcie, {
      			provide: Mapper,
		      	useValue: MockMapperRepository
		      }],
        }).compile();

        userService = moduleRef.get<UserServcie>(UserServcie);
        userController = moduleRef.get<UserController>(UserController);
    });

    describe('findOne', () => {
        it('should return one user array', async () => {
            jest.spyOn(userService, 'getUser').mockResolvedValue(user);
        
            expect(await userController.getUser("test")).toEqual(user);
        });
    });

    describe("createOne", ()=>{
        it("make one user", async ()=>{
            jest.spyOn(userService, "createUser").mockResolvedValue(userCreationResult);

            expect(await userController.createUser(newUser)).toEqual(userCreationResult);
        })
    })
});
```

여가서 주의깊게 보아야 할 부분은

``` typescript 

...생략...

const MockMapperRepository = ()=>({
    mapper: jest.fn()
})

describe('UserController', () => {
    let userController: UserController;
    let userService: UserServcie;

    beforeEach(async () => {
      const moduleRef = await Test.createTestingModule({
          controllers: [UserController],
          providers: [UserServcie, {
      			provide: Mapper,
		      	useValue: MockMapperRepository
		      }],
        }).compile();

        userService = moduleRef.get<UserServcie>(UserServcie);
        userController = moduleRef.get<UserController>(UserController);
    });

...생략...

```

이 부분이다.

Test의 createTestingModule 메소드를 통해서 테스트용 모듈을 생성하는 부분에서, Mapper 프로바이더를 바로 호출하여 생성하는 것이 아니라

``` typescript
{
  provide: Mapper,
  useValue: MockMapperRepositoy
}
```
와 같은 식으로 Mapper 프로바이더를 대체시켜 모듈을 생성해 준다.
이 때에, MockMapperRepositoy는 위에서 볼 때에 

``` typescript
const MockMapperRepository = ()=>({
    mapper: jest.fn()
})
```
가 있는데, Mapper 프로바이더의 mapper 메소드를 jest.fn()으로 모킹시키고, 클로져를 이용해 각 test case마다 독립된 값을 가질 수 있도록 함수로 작성한다.

그래서 결과는...?


### 결과
일단 DB에 접근이 없어졌고, 테스트 수행 시간이 짧아졌으며, 무엇보다 쓰레기 데이터가 디비에 쌓여서 일일히 청소해줘야 하는 문제가 사라졌다

간단한 문제인데 할줄을 모르는데다 뭐 도큐를 찾아봐도 안나오니 일단 삽질!


그리고 문제 해결! 끝!