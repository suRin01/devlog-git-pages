---
layout: post
title: "static과 singleton 차이"
subtitle: "한개긴 한갠데 두가지에요"
date: 2021-12-04 12:30:15 +0900
background: '/img/posts/05.jpg'
---


유틸리티 클래스를 만들던 중에, 싱글톤 패턴을 활용한 logger와 스태틱 메소드를 사용한 requestUtiliy 클래스 2가지를 만들게 되었다. 
그으....런데 싱글톤 클래스에서도 getInstance()가 스태틱으로 선언되어 있기도 하고 언제 스태틱 메소드로만 구성된 클래스를 써야하고 싱글톤 클래스를 써야할지 감이 안잡힌다.
스태틱 메소드를 이용한 클래스와 싱글톤 패턴을 활용한 클래스 2가지에 대해서 조금 찾아보자.

사실상 스태틱 메소드에 대한 글이 될 것 같은데, 한번 들어가봅시다


## 정적 매소드(static method)란 뭔가?
일단 정적메소드는 클래스의 인스턴스 없이도 호출할 수 있는 메도스이며, 인스턴스화 시킨 객체에서는 호출 할 수 없다.
이 장점 때문에 유틸리티 함수를 만드는데 유용하게 사용되고는 하는데, 객체를 만들고 연결하지 않아도 메소드를 호출할 수 있기 때문이다.
이 부분을 싱글톤과 구분되..는 점이라기 보다는 이러한 특성이 정적 메소드의 특성이라고 표현해야 하지 싶다.
이것을 코드를 통해 살펴보자면,

``` typescript
class A {
    public static method1(){
        console.log("method1 !");
    }



}

A.method1();
```
A란 클래스를 생성하고 스태틱 메소드 method1를 생성했고, 이 메소드는 클래스 이름 A를 인스턴스화 시키지 않고 접근할 수 있다.
객체를 만들어야 할 필요성이 없고, 인풋에 대한 특정한 처리가 필요할 뿐인 유틸리티 함수들이 이러한 점에서 장점을 가져가기 때문에 많은 유틸리티 클래스들이 정적 메소드로 선언되어서 사용되고는 한다.

또한 스태틱으로 선언된 프로퍼티는 스테틱 메모리(스택)에 올라가게 되고, 이 스테틱 프로퍼티는 스테틱 메소드에서만 접근할 수 있다. 이러한 스테틱 프로퍼티는 보편적으로 상수들만 모아서 사용하는 클래스로 많이 사용하게 되고, 예를 들어서 DB query string을 모아놓는다던가, 필요하다면 민감하지 않은 환경변수 등을 저장하는데 사용할 수 있을 것이다.

스태틱 프로퍼티는 스태틱 메소드에서만 접근할 수 있음을 나타내는 예제는 다음과 같다.

``` typescript
class A {
    public static method1(){
        this.property1 = 2;
        console.log(this.property1);
        // this.property2 = 1; // error
    }
    public method2(){
        // this.property1 = 3; // error
        this.property2 = 1; 
        console.log(this.property2);
    }
    public static property1 = 1;
    public property2 = 2;

    public constructor(){}


}

A.method1();

const a = new A();
a.method2();
```

또한, 스태틱 프로퍼티가 자주 사용되는 형태는 다음과 같다.

``` typescript
class QueryString{
  public static findAll = "Select * from `db`.`table`";
  public static createOne = "INSERT INTO `db`.`table` (`name`, `id`, `email`) VALUES (?, ?, ?)";
  //...
}

console.log(QueryString.findAll)
```



### 싱글톤 패턴
싱글톤 패턴은 프로그램 내에서 하나의 인스턴스만이 생성되고 재사용하기 위해서 고안된 패턴이다.
이러한 패턴은 logger, db conenction 등의 인스턴스를 생성하는데 유용하고, 하나의 인스턴스로 여러 클래스에서 공유하는 경우에 유용하다. 
생성자를 외부에서 호출할 수 없게 private로 설정하고, 외부에서는 getInstance() 메소드를 사용해 자기자신에 대한 인스턴스를 하나 만들어 외부에 제공하고, 이미 static으로 생성된 자기 자신의 인스턴스가 있다면 그 객체를 재활용해 반환하게 된다.

타입스크립트에서의 싱글톤 패턴 코드는 다음과 같다.

``` typescript

export class Logger {
  
	private static instance: Logger;
	private constructor() {}

	public static getInstance(): Logger {
		if (this.instance) {
			return this.instance;
		}
		this.instance = Logger();

		return this.instance;
	}
}

```

## 정리
typescript에서는 static class는 존재하지 않는다.
하지만 static method와 static property는 존재한다.
static method와 property는 프로그램 시작 시에 메모리를 할당시켜주게 되고, 싱글톤 패턴은 이러한 원리를 조금 더 발전시켜서 기존에 property로 할당시켜놓은 주소에 인스턴스를 저장하고 활용하게 된다.
결국 static과 singleton은 차이점으로 구분지어야 할 개념이 아니라, 기본 개념을 발전시켜서 활용한 개념으로 보아야 하는것이 맞지 않나 하는 생각이 든다.