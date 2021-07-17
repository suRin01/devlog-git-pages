---
layout: post
title: "단순 복사와 얕은 복사와 깊은 복사"
subtitle: "복사 방법들"
date: 2021-07-17 12:30:15 +0900
background: '/img/posts/05.jpg'
---


# 단순 객체 복사와 shallow copy와 deep copy

## 단순 객체 복사
Javascript에서 객체를 연산자로 대입하게 되면, 대입 연산자는 원본 객체의 주소를 할당시킨다. 결국 대입된 객체와 원본 객체 모두 하나의 주소를 가르키고 있기 때문에, 어느 변수를 통해 접근하든 같은 객체를 건들고 있는 것과 같다.

``` javascript 
let case1 = {a:10, b:20, c: {d:30, e:40}};
let case2 = case1; // 단순 객체 복사
case2.c.d = 100;
console.log(case1.c.d); // expected output: 100

```
와 같이 된다.
이는 위에서 말한 것 처럼, case1과 case2 모두 하나의 주소를 가르키고 있다.

## shallow copy(얕은 복사)
Object의 assign 메소드를 이용하여 복사를 할 때, 복사되는 원본 객체 첫 depth의 복사가능한 모든 프로퍼티를 복사한다. 그런데 이 방법을 통해서 객체 복사를 진행하게 될 때, 발생되는 문제점이 몇 가지가 있다. 이 점들을 예제를 통해서 볼 때에, 

### Property Descriptors 복사 불가
``` javascript 
let case1 = {a:10, b:20, c: {d:30, e:40}};
Object.defineProperty(case1, "a", {writable: false});
Object.getOwnPropertyDescriptor(case1, "a");
// {value: 10, writable: false, enumerable: true, configurable: true}

let case2 = Object.assign({}, case1);
Object.getOwnPropertyDescriptor(case2, "a");
// {value: 10, writable: true, enumerable: true, configurable: true}

```
위의 예제에서 assign()을 통해서 복사된 객체는 descriptor는 복사되지 않음을 볼 수 있다. 

### Prototype Chain이나 열겨가능한 프로퍼티 
``` javascript
const protoObject = {
    a : 10
}
let case1 = Object.create(protoObject, {
    b:{
        value: 20
        //enumerable-> default : false
    },
    c:{
        value: 30,
        enumerable: true
    }
});

let case2 = Object.assign({}, case1)
console.log(case2); // {c:30}
```
Object.create로 생성한 객체에서 프로퍼티의 기본 열거 가능은 불가로 설정되어 있고, 이렇게 생성한 객체를 assign으로 얕은 복사를 통해 복사할때 enumerable을 true로 설정하지 않을 경우 복사되지 않는다. 또한, case2의 __proto__는 case1의 __proto__가 {a:10}으로 되어이있는 반면, Object에 바로 연결 되어 있다. 프로토타입 체인까지 복사되지 않는다는 것이다.

이러한 문제점들을 해결하기 위해서 우리는 deep copy, 깊은 복사를 통해서 복사를 진행하게 되는데 깊은 복사는 얕은 복사에서 주소를 복사하는 것 처럼 복사를 진행하는게 아니라 새로운 메모리 공간을 확보하고 그곳에 데이터를 복사하게 된다. 

javascript에서 진행할 수 있는 방법으로는
1. 재귀를 통한 복사
2. JSON라이브러리를 통한 복사
3. 타 라이브러리(jQuery, lodash)등을 이용한 복사

가 있다. 
이 중 첫 재귀를 통한 복사로서 

``` javascript 
function deepCopy(obj) {
  if (obj === null || typeof(obj) !== "object") {
    return obj;
  }
    
  let copy = {};
  for(let key in obj) {
    copy[key] = deepCopy(obj[key]);
  }
  return copy;
}

```
와 같은 함수를 만들어서 사용할 수 있겠고,

JSON 라이브러리를 통한 복사는
``` javascript

let copy = JSON.parse(JSON.stringify(origin));

```
와 같은 방법으로 진행할 수 있다.

타 라이브러리를 사용할 때에는
``` javascript 

// jQuery
let copy1 = $.extend(true, {}, origin);
// lodash
let copy2 = _.cloneDeep(origin);

```
의 방법으로 사용하면 되겠다.

하지만 1의 방법이나 3의 방법에 비해서, 2의 방법은 성능 차이가 있으니(약 10배 이상) 가급적 1의 방법이나 3의 방법을 사용하기를 권한다.



# 결론
JS에서 원시 타입을 제외한 모든 것들은 객체이기 때문에 복사를 할 때에는 조심해야 한다는 생각이 들었다. 잘못 복사하다가 진짜 서로 데이터 다 얽혀서 망하는 꼴을 보고 싶지 않다면 말이다.
근데 이정도면 Object에서 메소드로 deepCopy를 넣어줄 만도 한데 왜 안넣어주는거지...?
