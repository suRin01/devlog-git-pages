---
layout: post
title: "typescript에서 interface와 class 차이"
subtitle: "지들이 특별한 줄 알아요"
date: 2021-11-13 12:30:15 +0900
background: '/img/posts/05.jpg'
---



# interface, class
거의 모든 객체 지향 언어에서 인터페이스를 지원하고, 그 인터페이스를 바탕으로 클래스를 설계하고 구현하고 인스턴스화 해서 사용한다.
지금 진행중인 프로젝트에서 데이터 타입을 클래스로 선언하고 리턴 형식을 명시하는데, 이 때에 인터페이스와 클래스 둘 중 무엇이 더 나은지에 대해서 궁금해졌다.
이러한 것들에 대해서 찾아보고 개인적으로 낸 결론에 대해서 정리해보고자 한다.

클래스와 인터페이스에 대한 기초적인 내용은 숙지하고 있을것이라는 가정하에, 타입스크립트에서 어떻게 소개하고 어떠한 방식으로 사용하는지에 대해서만 짚고 넘어간다.

## class
클래스란 객체 지향 프로그래밍(OOP)에서 특정 객체를 생성하기 위해 변수와 메소드를 정의하는 일종의 틀이다. 객체를 정의하기 위한 상태(멤버변수)와 메소드(함수)로 구성된다[링크](https://en.wikipedia.org/wiki/Class_(computer_programming)) 
타입스크립트는 ES2015에서 소개된 class 키워드를 지원하고 있다. 자바스크립트에서 제공하는 기능과 다르게, 타입스크립트는 다른 클래스와 타입들 간의 관계를 표현하기 위한 어노테이션이나 기타 문법들을 추가하였다.

모든것이 객체로서 이루어진 자바스크립트에서, 클래스는 객체 팩토리로서 작동하게 된다.


## interface
인터페이스는 ES6에서 지원하지 않는 타입스크립트만의 특징이다. 인터페이스는 타입이고, 컴파일후에 존재하지 않는다. 인터페이스는 선언만 존재하며, 멤버 변수와 메소드를 선언할 수 있지만 접근제한자는 설정할 수 없다.

클래스와 달리 inteface는 타입스크립트의 컨택스트에서만 존재하는 가상 구조이고, 타입스크립트 컴파일러는 타입 체크 목적으로만 인터페이스를 사용하게 된다.
결국 자바스크립트로 트랜스파일 될 때 인터페이스는 제거된다는 뜻이다.

타입스크립트의 주 목적인 타입을 설정하고 예측할 수 있는 리턴 타입, 예기치 못한 개발자의 실수를 줄이기 위한 방법으로서 함수에 전달하는 인자의 형식을 고정할 때 사용한다고 할 수 있다.

이 뿐만 아니라, 추상클래스로서도 사용될 수 있다. 메소드의 형태만 선언해서 인터페이스를 정의하고, 이후에 클래스를 정의할때 implements 키워드를 통해 이 인터페이스를 지정하면 이 클래스는 추상함수로 선언된 메소드의 몸체를 구현해야 한다. 

### abstract class
그럼 abstract class가 나온 이유는 무엇인가?
누군가는 abstract class를 interface로 대체하는것이 좋다는 의견 또한 본 적이 있다.
그럼 타입스크립트에서 abstract class가 interface와 다른점을 찾아보고, 어떠한 목적으로 이러한 문법을 도입했는지 가늠해보자.

추상클래스의 기본적인 형태는 다음과 같다

``` typescript
abstract class job{
    readonly nickname: string;
    constructor(nickname: string){
        this.nickname = nickname;
    }

    greet(): void{
        console.log(`My name is ${ this.nickname }.`);
    }

}
```

위의 예제를 볼때, 타 언어들의 abstract class와 다르게 메소드에 대해 구체적 구현이 가능함을 알 수 있다.
하지만, job이라는 abstract class를 가지고 객체를 생성하는 것은 불가능하다.

위의 abstract class를 가지고 활용한 예제를 하나 보도록 하자.

``` typescript
abstract class Job {
    readonly nickname: string;
    constructor(nickname: string) {
        this.nickname = nickname;
    }
    greet(): void {
        console.log(`My nickname is ${ this.nickname }.`);
    }
    abstract attack(): void;
}
class Warrior extends Job {
    attack() {
        console.log('검을 사용해 공격!');
    }
}
class Magician extends Job {
    attack() {
        console.log('마법을 사용해 공격!');
    }
}

const warrior: Job = new Warrior('heecheolman');
const magician: Job = new Magician('heecheol');
warrior.greet(); // My nickname is heecheolman
magician.greet(); // My nickname is heecheol
warrior.attack(); // 검을 사용해 공격!
magician.attack(); // 마법을 사용해 공격!
```

예제에서 볼 수 있듯, 추상 클래스에서 구현된 greet이라는 메소드가 하위 구현 클래스의 객체에서 정상적으로 사용되는 것을 볼 수 있다.
추상 클래스는 구현 메소드와 추상 메소드가 동시에 존재할 수 있다는 말이다.
또한 추상 메소드는 추상 클래스와 같이 abstract 키워드를 사용하여 추상메소드를 선언하여야 하고, 이렇게 선언된 추상 메소드는 이하 구현 클래스에서 구현되어야한다.


그렇다면 추상클래스는 왜 사용하는 것일까?
인터페이스와 추상 클래스를 비교한 점들을 한번 보도록 하자.[링크](http://dotnetpattern.com/typescript-abstract-class)

인터페이스 
- 모든 멤보가 추상적
- 인터페이스는 다중 상속을 지원
- 타입스크립트 인터페이스는 자바스크립트로 트랜스파일되지 않고 타입스크립트 코드에서만 사용 가능함

추상 클래스
- 몇몇 멤버는 추상적일 수 있고 몇 멤버는 완전히 구현될 수 있다.
- 추상 클래스는 다중 상속을 지원하지 않는다.
- 추상 클래스는 자바스크립트 기능으로 트랜스파일된다.

결국 인터페이스와 추상 클래스 두가지 모두 사용해야 할 경우가 생기게 된다.

``` typescript
interface IName {
    firstName: string
    lastName: string;
}
 
interface IWork {
    doWork(): void;
}
 
abstract class BaseEmployee implements IName, IWork {
    firstName: string;
    lastName: string;
 
    constructor(firstName: string, lastName: string) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
 
    abstract doWork(): void;
}
 
class Employee extends BaseEmployee {
    constructor(firstName: string, lastName: string) {
        super(firstName, lastName);
    }
 
    doWork(): void {
        console.log(`${this.lastName}, ${this.firstName} doing work...`);
    }
}
 
let emp: IWork = new Employee('Dana', 'Ryan');
emp.doWork();
```

위의 예제를 볼 때, BaseEmployee는 Iname, IWork 2가지 인터페이스를 상속받고, absctract로 선언한다.
이 BaseEmployee를 Employee가 상속받아 구현하게 되는 모양이다.

이러한 경우 인터페이스가 추상메소드의 자리를 대체하건, 추상메소드가 인터페이스의 자리를 대체하는 방법은 없게 된다.

각자의 역할이 있는 것이다.


### 그래서 interface랑 class중 뭘 써야하는데요?
타입스크립트 공식 문서에서 데이터 타입을 선언할 때 인터페이스를 사용하기를 권장한다.
하지만 더 찾아봤을때, 이것 또한도 케이스 바이 케이스라는 말이 있다. 
반환하는 객체를 조작하는 로직이 필요할 경우, 예를 들어서 ExecutionResult라는 db에서 받아온 데이터를 처리하여 객체로 반환하는 경우 이러한 것들을 string으로 바꿔주는 toString()이라는 메소드가 있다면 편리하게 사용할 수 있을 것이다.
이런 경우가 있기 때문에 케바케라는 말이 나오는 것이다.


나의 경우, 이러한 작업은 아직 필요하지 않지만 작업 결과를 winston등을 사용하여 로깅하고 콘솔로 출력할때 스트링으로 바꿔주는 작업이 필요하게 될 수도 있다. 이러한 경우를 위해서 인터페이스가 아니라 클래스로 선언하고 내부적으로 toString() 메소드를 구현해 놓으면 좋겠다는 생각이 들었다.


결국 best practice는 존재하지만, 모든 경우에 맞는 유일한 해답은 없는것 같다.