---
layout: post
title: "ProtoType에 대해서"
subtitle: "JS에서 객체가 생성되는 방법"
date: 2021-07-02 12:30:15 +0900
background: '/img/posts/05.jpg'
---
# Prototype
## prototype?
javascript에서 객체를 상속하기 위해서 사용하는 방식.
프로토타입 체인을 통해서 상위 프로토타입 객체로부터 메소드와 속성을 상속받고, 사용할 수 있도록 함.
(상속이라고 표현하지만, 어떤 객체를 원형으로 삼고 이를 복제(참조)함으로써 상속과 비슷한 효과를 냄)

JS에서 모든 객체는 자신을 생성한 생성자의 prototype 프로퍼티가 가르키는 개체를 자신의 프로토타입 객체(부모 객체)로써 취급한다.

일단 프로토타입에 들어가기 앞서, 자바스크립트에서 객체를 어떻게 생성 하는지에 대해서 알아보자.

### 객체 리터럴 방식

```javascript
let factory = {
	name : 'foo',
	number : '00010'
}
```
이러한 방식으로, 중괄호를 이용해서 객체를 생성하는 것이 객체 리터럴 방식이다. 중괄호 안에 아무것도 기술하지 않는다면 빈 오브젝트가 생성이 되고, 안에 프로퍼티 이름:프로퍼티 값 형태로 표기하면 해당 프로퍼티가 추가된 객체를 생성할 수 있다. 이 때 프로퍼티 값이 함수일 경우를 메소드라고 한다.

### 생성자 함수 이용

```javascript
function Factory(name, number) {
    this.name = name;
    this.number = number;
}

```
자바스크립트에서는 c++나 자바와 같은 객체지향 언어에서의 생성자 함수의 형식과는 다르게 기존 함수에 new연산자를 붙여서 호출하면 해당 함수는 생성자 함수로 동작한다. 그래서 대부분의 스타일 가이드에서는 위의 코드와 같이, Factory의 F를 대문자로 쓰기를 권장한다.


### 객체 리터럴 방식과 생성자 함수 방식의 비교
객체를 생성할때 주로 사용하는 객체 리터럴 방식과 생성자 함수 방식으로 생성했을때, 가장 큰 차이점은 재사용의 문제다. 당연히 생성자 함수를 만들어놓고 사용하는것이 여러개의 객체를 다른 프로퍼티 값으로 생성할 때 편리할 것이다.

그 외적으로 중요한 차이점을 보자면, 

![리터럴방식](/img/posts/21_07_02/1.png "이미지 설명(title)")

상기 이미지에서는 객체 리터럴 방식으로 객체를 생성하였다. 이때 __proto__라는 히든링크를 통해서 상위 프로토타입에 연결되어있고, literal.hasOwnProperty라는 메소드를 호출한다면 일단 해당 literal에는 hasOwnProperty가 없기 때문에, __proto__를 통해서 상위 객체인 Object에서 hasOwnProperty가 있는지 확인한다. 

하단의 console.dir(Object.prototype)을 했을때 나오는 메소드들과, literal에서 __proto__를 통해서 타고 올라갔을때 나오는 메소드들이 같음을 알 수 있다.



![생성자 함수 방식](/img/posts/21_07_02/2.png "이미지 설명(title)")

상기 이미지에서는 생성자 함수 방식으로 객체를 생성하였다. 이때 바로 __proto__라는 히든링크로 Object에 연결되는것이 아니라, Factory.prototype으로 연결이 되고 거기서 Object.prototype으로 연결이 되는것을 확인할 수 있다.


이와 같이 __proto__라는 히든 링크로 프로퍼티들이 연쇄적으로 이어진 것을 프로토타입 체인이라고 하고, 이 체인을 따라 검색하는 것을 프로토타입 체이닝이라고 한다.


결국, 어떠한 생성자 함수이든 prototype은 객체이기 때문에 Object.prototype이 언제나 프로토타입 체인의 최상단에 존재하게 된다.


## 결론

자바스크립트에서의 유일한 생성자는 객체이고, 각각의 객체는 자신의 프로토타입이 되는 다른 객체를 __proto__를 통해서 가르킨다. 또한 그 객체도 프로토타입을 가지고 있고 이것이 계속해서 반복하게 되고 이것을 이용해 프로토타입 기반의 언어인 자바스크립트에서 속성, 메소드 상속을 구현하게 된다. 

이렇게 구현된 기능들은, ECMAScript 2015에서 몇가지 키워드와 함께 class를 구현하게 된다. 이런 생성 방식은 기존 클래스 기반 언어 개발자들에게 친숙하지만, 동작방식이 완전히 같이 않고 여전히 프로토타입 기반으로 구현되게 한다.

```javascript
class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

let square = new Square(2);
```

위 예제를 봤을때, polygon이라는 클래스를 생성하고 square는 polygon이라는 클래스를 상속받는다. 이때 생성자 constructor(sideLength){...}에서 super를 통해 부모 오브젝트의 생성자 함수를 호출하고, super(sideLenth, sideLength)를 통해 생성자를 구현한다. 이후 메소드들을 추가하고, 아랫줄에서 새 클래스를 사용하는 과정까지의 코드이다. 

이는 실제 콘솔에서 어떻게 돌아갈까?


![클래스](/img/posts/21_07_02/3.png "이미지 설명(title)")

다음과 같이, Square로 생성된 square는 프로토타입 체인을 봤을때 상위 polygon과 object까지 체이닝이 되어있는 모습을 볼 수 있다. 
결국 자바스크립트에서 클래스를 구현한 것은 기존의 자바스크립트가 프로토타입 기반의 언어라는 것을 해치지 않았고, 프로토타입 체이닝을 사용하여 구현한 클래스의 경우 계속해서 체인을 타고 들어가며 검색하여야 하기 때문에 올바르지 않은 설계일 경우 성능상의 문제가 나올 수도 있을 것 같다.

추가로 더 읽어볼만한 MDN의 링크를 남긴다 [[링크]](https://developer.mozilla.org/ko/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)