---
layout: post
title: "var, let, const"
subtitle: "부제: var의 문제점"
date: 2021-05-13 02:13:13 +0900
background: '/img/posts/05.jpg'
---
<h2 class="section-heading">var, let, const</h2>
<p>일단 결론부터 내자.</p>

<p>    var, let, const의 차이점</p>
<p></p>
<p>    1. var 함수 레벨 스코프 / let, const 블럭 레벨 스코프</p>
<p>    2. var로 선언한 변수 선언 이전에 사용 가능 / let, const 사용 불가 -> Temporal Dead Zome(TDZ, https://stackoverflow.com/questions/33198849/what-is-the-temporal-dead-zone/33198850#33198850)</p>
<p>    3. var 재선언 가능 / let, const 재선언 불가</p>
<p>    4. var, let 선언시 초깃값 할당 불필요 / const 초깃값 할당 필수(const니까)</p>
<p>    -> var, let 값 재할당 가능 / const 할당한 값 변경 불가(객체 안에 프로퍼티가 변경되는건 가능)</p>
<p>    -> const object의 object.freeze() 라는 방법도 있지만, 얕은 동결(https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)</p>
<p></p>
<p>    그럼 let과 const는 호이스팅이 발생하지 않는가? 발생한다.</p>
<p>    그렇기 때문에 TDZ과 같은 문제가 생길수 있는 것이다.</p>
<p>    (그렇다고 TDZ가 나쁜건 아니다. 선언 이전에 접근하려고 하는 등의 문제를 막을수 있기 때문.)</p>
<p></p>
<p></p>

<p></p>
<p></p>
<h2 class="section-heading">아니 그래서 뭐 어떤걸 쓰라고?</h2>
<p><p>let과 const를 애용하는것이 정신 건강에 이롭다.</p></p>
<p>var의 문제점이 뭔데?</p>

<p>var가 좋고 쓸만했으면 굳이 let과 const를 만들 이유가 뭔가? 다 문제가 있어서 그런 것이다.</p>
<p>4개의 예를 들어 설명해보자. 위에 나온 것들 또한 있다</p>
<p></p>
<p>1. 재선언</p>
<p>    일단 쉽다. let으로 선언한 변수는 재선언이 불가능하다. </p>
<p></p>
<p></p>
<p>2. 변수의 스코프와 lifeTime 문제</p>
<p>function a(){</p>
<p>    console.log(x) // undefined</p>
<p>    {</p>
<p>        var x = 10;</p>
<p>        console.log(x) // 10</p>
<p>        let y = 20;</p>
<p>    }</p>
<p>    console.log(x) // 10</p>
<p>    console.log(y) // ReferenceError: y is not defined</p>
<p>}</p>
<p>    애가 막 스코프를 뚫고 미쳐 날뛴다. </p>
<p></p>
<p>    우리는 당연하게(다른 언어를 배워왔듯) 스코프 단위로 lifeTime을 예상하고 그에 맞게 행동하는 것에 익숙해져 있다. </p>
<p>    이렇게 스코프를 무시하고 선언 후에 라이프타임이 종료되지 않는 변수는 버그를 일으키는 주범이 될 뿐이다. </p>
<p></p>
<p>3. 호이스팅</p>
<p>    이전에서의 설명과, 2번의 예제를 통해 동시에 알 수 있다.</p>
<p>    var, let, const는 모두 호이스팅 된다. </p>
<p></p>
<p>    하지만 var는 함수 레벨 안 선언 이전에서 접근 가능하고, 그 결과로 a함수의 첫 라인에서 x를 콘솔로 찍었을때 undefined로 출력된다. </p>
<p></p>
<p></p>
<p>4. 클로저</p>
<p>    closure를 직역하면 폐쇄라는 뜻이다. </p>
<p>    뭘 위한 무엇을 폐쇄라는 건가?</p>
<p>    </p>
<p>    예제를 하나 가져왔다.</p>
<p>    function makeAdder(x){</p>
<p>        let y = 1;</p>
<p>        return function(z){</p>
<p>            y = 100;</p>
<p>            return x+y+z;</p>
<p>        };</p>
<p>    }</p>
<p></p>
<p>    let add5 = makeAdder(5);</p>
<p>    let add10 = makeAdder(10);</p>
<p></p>
<p>    console.log(add5(2)); //107, x:5, y: 100, z: 2</p>
<p>    console.log(add10(2)); //112, x:10, y: 100, z: 2</p>
<p></p>
<p>    add5와 add10은 둘 다 클로저이다. 클로저는 외부 변수를 기억하고 그대로 가져와서 폐쇄시킨다. </p>
<p>    자바 스크립트에서는 new function을 제외하고는 모든 함수가 자연스럽게 클로저가 된다.(https://ko.javascript.info/closure)</p>
<p>    함수는 숨김 프로퍼티인 [[Enviroment]]를 이용해서 자신이 어디서 만들어졌는지를 기억하고, 함수 내부의 코드는 [[Enviroment]]를 이용해서 외부 변수에 접근한다.</p>
<p>    모든 것에는 다 이유가 있는 법인가 보다. </p>
<p></p>
<p>    아무튼 이 클로저가 왜 var를 사용했을때 문제가 생기냐면</p>
<p></p>
<p>    var functions = [];</p>
<p>    for (var i = 0; i < 3; i++) {</p>
<p>        functions[i] = () => { console.log(i); };</p>
<p>    }</p>
<p>    for (var j = 0; j < 3; j++) {</p>
<p>        functions[j]();</p>
<p>    }</p>
<p>    // 3 3 3</p>
<p></p>
<p>    어? 어데서 자주 보던 문제다(원인은 다르다)</p>
<p>    위의 코드에서 i로 클로저가 연결되어있다.</p>
<p>    for를 통해 3개의 클로저가 생성되지만 하나의 자유변수 i를 공유하기 때문에 발생하는 일이다.</p>
<p></p>
<p>    위의 해결 방법은</p>
<p>    var functions = [];</p>
<p>    for (var i = 0; i < 3; i++) {</p>
<p>        (function(i){</p>
<p>            functions[i] = () => { console.log(i); };</p>
<p>        })(i)</p>
<p>        </p>
<p>    }</p>
<p></p>
<p>    와 같은 형식으로 클로저를 하나 더 생성하면 되지만.... </p>
<p>    그럴 필요가 있을까?</p>
<p></p>
<p>    for (let i = 0; i < 3; i++) {</p>
<p>        functions[i] = () => { console.log(i); };</p>
<p>    }</p>
<p></p>
<p>    이렇게만 바꿔줘도 해결이 되는 문제다. </p>
<p></p>
<p>    var를 굳이 쓸 필요가 없다</p>
<p></p>
<p>+  전역 객체 문제</p>
<p>    node.js에서와 웹 환경 상에서 this가 가르키는 객체는 서로 다르다. </p>
<p>    간단하게 말해서 웹에서 this는 window를 가르키고, node.js에서는 module.exports를 가르킨다. </p>
<p>    이는 다음에 더 집중적으로 공부해 보도록 하고, 본론으로 들어가서 this가 왜 갑자기 튀어나왔는지를 알아보자. </p>
<p></p>
<p>    웹 환경의 탑 레벨에서</p>
<p>    var x = 0;</p>
<p>    let y = 0;</p>
<p>    일 때,</p>
<p></p>
<p>    console.log(window.x);  // 0</p>
<p>    console.log(window.y);  // undefined</p>
<p></p>
<p>    var로 선언해버리면 글로벌 오브젝트 window의 프로퍼티에 추가되어 버린다. </p>
<p>    전역으로 붙어버리면 안좋은 이유는...</p>
<p>    웹상에서는 소스와 데이터가 클라이언트에 공개되고, 비동기 로직이 비교적 쉽게 구현되고, 모바일의 퍼포먼스가 치고 올라왔다 하더라도 여전히 pc에 비해서는 딸리는게 사실이다.</p>
<p>    이러한 환경에서 전역 변수가 추가되고 많이 사용되다 보면 결국 손해를 볼 수 밖에 없다. </p>
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<h1 class="section-heading">결론.</h1>
<p></p>
<p>    var의 시대는 갔고 앞으론 let과 const의 시대다. </p>
<p></p>
<p>    let과 const는 이미 15년 6월에 es6라는 이름으로 업데이트 되었다. </p>
<p>    6년이 거진 다 되어가는데 지금 와서 과거의 그리 찬란하지 않은 유산에 얽매일 필요는 없다고 생각된다. </p>
<p></p>
<p>    새 것을 배우자</p>
<p></p>
<p></p>