---
layout: post
title: "Duck typing?"
subtitle: "뭔가 마음에 안드는 방식의 코딩법. 사용하는 이유가 있을까?"
date: 2021-10-10 12:30:15 +0900
background: '/img/posts/05.jpg'
---


# Duck Typing

> If it walks like a duck and it quacks like a duck, then it must be a duck<br>
"오리처럼 걷고 오리처럼 소리낸다면, 그것은 분명 오리일것이다"

duck typing에 대해서 검색했을때 가장 많이 인용되는 문구일 것이다.

하지만 이러한 문구보다 더 기억에 남는건, 

![duck](/img/posts/21_10_8/pig.jpg)


이 사진이지 싶다(...)

아무튼 왜 갑자기 이런 주제를 가져와서 글을 쓰는가 하니, 타입스크립트에서 인터페이스와 클래스의 차이에 대해서 찾아보려고 오피셜 도큐[링크](https://www.typescriptlang.org/docs/handbook/interfaces.html)를 보다가, 처음보는 단어인 duck typing에 대해서 봤다.


대충 슥슥 읽어보니, 타입스크립트의 중요 원리중 하나가 값을 가지고 있는 모양에 초점을 맞춰 형검사를 하는 것이라고 한다. 이게 종종 덕 타이핑, 혹은 (완전히 동일한 개념은 아니지만,[링크1](https://wiki.c2.com/?DuckTyping), [링크2](https://wiki.c2.com/?DuckTyping)) 구조적 서브 타이핑이라고 하는데, 이게 명목적(nominal) 타이핑과는 대조적인 방식이라고 한다.

근데 나는 이때까지 덕 타이핑은 고사하고, 명목적 타이핑이라는 단어도 아직 못 들어본것 같다.

그럼 덕 타이핑에 들어가기 전에, 명목적 타이핑에 대해서부터 알아보자.



## 명목적 타이핑(Nominal Typing)
명목적 타이핑은 두개의 변수가 같은 이름의 자료형으로 선언된 경우에만(if and only if) 호환된다.
예를 들어 c언어에서, 두 구조체가 같은 멤버를 가지고 있다고 하더라도 이름이 다른 경우에 호환되지 않는다.
물론 c에서 typedef을 통해서 이미 존재하는 자료형의 별칭을 선언할 수 있다. 하지만 이것은 문법적 요소일 뿐, 자료형 체크를 위해 별칭과 다른 이름을 구분할 수는 없다. 이 기능은 자료형 안전성에 문제를 일으킬 수 있고, 만약 하나의 int 자료형은 두개의 의미로 다른 방식을 통해 사용되는 경우 문제가 된다.


## 명목적 서브타이핑(Nominal SubTyping)
명목적 서브타이핑은 위의 명목적 타이핑에서, 한 자료형이 다른 자료형의 서브타입이 되도록 명시적으로 선언한 경우, 그 자료형은 다른 자료형의 서브타입으로 인정되게 된다. 이 방식은 c++, java 등에서 채용되며, java 코드를 통해 보았을 때 다음과 같은 형태가 된다.

``` java
class Person {
    public name: string;
}

class Employee extends Person {
    public salary: number;
}

class Manager extends Employee { }

class Product {
    public name: string;
    public price: number;
}
```
위의 코드에서, employee는 person의 서브타입이고, manager는 employee의 서브타입이다.
반면에, product는 그 어떤 타입을 extends하지 않기 때문에, 위의 세 타입과 구분된다.


## 덕 타이핑(Duck Typing)
그럼 덕 타이핑은 무엇인가? 
위에서 서로 다른 두 타입은 구분되고, 상속을 통해서 서브타이핑을 구현한다. 
하지만 덕 타이핑은, 좀 더 이상한 일을 하게 된다.

일단 코드를 통해 볼 때, 

``` javascript
 let duck = {  
    appearance: "feathers",  
    quack: function duck_quack(what) {  
        print(what + " quack-quack!");  
    },  
    color: "black"  
};

let someAnimal = {  
    appearance: "feathers",  
    quack: function animal_quack(what) {  
        print(what + " whoof-whoof!");  
    },  
    eyes: "yellow"  
};  

function check(who) {  
    if ((who.appearance == "feathers") && (typeof who.quack == "function")) {  
        who.quack("I look like a duck!\n");  
        return true;  
    }  
    return false;  
}  

check(duck);  // true
check(someAnimal);  // true
```

맨 처음 시작할 때, "만약 어떤 동물이 오리처럼 보이고, 오리처럼 소리낸다면 그것은 분명 오리이다."라는 문구를 써놓았다.

위의 코드는 그 문구를 그대로 코드를 구현한 모습니다.

check라는 함수는 인자로 받아들인 who의 외형이 feather를 가지고(오리처럼 보이고), quack이라는 이름을 멤버가 함수를 나타내고 있다면 true를 반환한다.

조금 더 나아가서, 이 코드도 한번 보도록  하자.


``` javascript
interface Quackable { quack(): void } 
class Duck implements Quackable { 
	quack() { 
		console.log('Quack'); 
	} 
} 
class Person { 
	quack() { 
		console.log('Quack'); 
	} 
} 
function inTheForest(quackable: Quackable): void { 
	quackable.quack(); 
}
	
inTheForest(new Duck()); // OK 
inTheForest(new Person()); // OK

```


위의 예제에서, duck은 quackable이라는 인터페이스를 상속받아 quack를 구현하고 있다.

반면, person은 어떠한 인터페이스를 상속받지 않는다.

그런데 inTheForest 함수에서 인자로 받는 quackable은 타입을 Quackable로 명시하고 있는데, 아래의 inTheForest(new Person());의 코드는 정상적으로 작동하는 것을 볼 수 있다.

이는 Person 클래스가 quack이라는 함수를 구현하고 있고, 이는 덕 타이핑의 중요 관점인 객체가 하나가 값을 가지고 있는 모양에 초점을 맞춰 형검사를 하는 것을 만족하기 때문에 정상적으로 작동하는 것이다.


## 덕 타이핑의 장점
덕 타이핑은 적은 코드로 상속 구조를 구현할 수 있다. 따로 인터페이스를 지정할 필요가 없고, 필요한 메서드를 작성하면 되기 때문이다. 이러한 점은 여러 클래스를 사용하고 상속이 중첩될 때 generic hell이 발새할 수 있는 상황을 사전에 제거하게 된다. 또한, 이 개념은 다향성이니 SOLID니 기타 여러 개념들보다 이해하기 쉽다. 그냥 인터페이스를 만들어놓고, 그 대로 구현만 하면 된다.


# 덕타이핑의 단점
이러한 방식이 물론 장점만 있는 것은 아니다. 이러한 방식의 구현은 하위 클래스로 만들 클래스가 인터페이스를 정확히 잘 지키고 있는지 컴파일, 혹 빌드 단계에서 찾기 힘들다. 그렇기 때문에 유닛 테스트로 해당 객체가 정상적으로 계속 체크해야 하고, 많은 사람들과 협업하게 될 때 주의가 필요하다.



### 마무리하면서
처음에 덕 타이핑과 명목적 타이핑의 차이는 타입에 대한 차이로 나오는 줄 알았다. 것도 그런게, 덕 타이핑에서 주로 예를 드는 언어는 자바스크립트, 파이썬 등 형에 민감하지 않은 언어들이기도 하고, 그 반면에 명목적 타이핑에서는 주로 예를 드는게 c언어, 자바 등의 언어니까 말이다.


하지만 타입스크립트 또한도 덕 타이핑을 지원하고, 조금 더 읽다 보니까 덕 타이핑과 명목적 타이핑의 차이는 형식에서 차이가 오는게 아니라, 인터페이스를 생성하고 어떠한 객체가 위에서 Quackable이라는 인터페이스 인자의 자리에 들어가서 정상적으로 동작하는것 처럼, 그 객체가 어떠한 일을 할 수 있는지 체크하고 할수 있기만 한다면 허용한다는 점은 꽤나 편리할 수 있을 것 같다.

이 점에 대해서 좋다 나쁘다를 나누기에는 애매할 것 같고(인터넷에 duck typing에 대한 글들을 찾아보면 맨날 싸우고 있다.) 타입스크립트 공식 도큐에서도 이러한 점에 대해서 별다른 말을 하지는 않기 때문에, 

개인적으로 지양하는 방식으로 다가오지만 이러한 방식 또한 있다 라는 것은 알아놓는게 좋을 것 같다.

이상 끗!
