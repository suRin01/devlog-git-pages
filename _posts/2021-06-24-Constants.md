---
layout: post
title: "Node.js에서의 상수"
subtitle: "Nodejs에서 상수 외부파일로 빼기"
date: 2021-06-24 12:30:15 +0900
background: '/img/posts/05.jpg'
---
# Constant를 써야 하는 이유
const로 저장되는 데이터들은 보통 특정 리소스(이미지, URL, 서버 주소 등등)의 링크 등에 쓰기 위한 스트링이다. 
이러한 것들을 코드 내에 직접적으로 입력해놓게 되면 하드코딩했다고 하면서 대차게 까이거나 하는게 대부분이다.  
개발 진행하다가 보면 이런 주소나 위치는 계속 바뀌고 그럴때마다 그 링크가 사용된 곳을 일일히 찾아줘야 하는 문제점이 발생하기 때문이다.
뭐 다 바꾸면 되지 그게 뭐가 문제냐? 하는 사람도 있겠지만, 막상 하나하나 바꾸다 보면 꼭 하나씩 빼먹고 안바꾸는 것들이 존재한다. 



## json in .js
```javascript
//constants.js
const  jsonVariable = {
	aa:  "bb",
	cc: {
		dd:  "ee"
	}
}
Object.freeze(jsonVariable)
module.exports = jsonVariable;

//use.js
let  constant = require("./constant")
console.log(constant.cc.dd); // expect output: ee
```
json으로 .js파일에 저장하고, require나 import로 불러와서 사용하는 방식이다.
간단하고 직관적이고 사용하기 쉽지만, 불러왔을때 객체의 값을 수정할 수 있는 문제점이 존재한다. 그 대응 방안으로 Object.freeze(Object)를 사용하여서 변수를 constant처럼 접근은 가능하지만, 값은 바꿀수 없도록 말 그대로 얼릴 수가 있다. 하지만 여기서 문제점이 발생한다.
Object.freeze(Object)는 얕은 동결(mdn(https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)으로서, 첫 직속 속성에만 적용되고 속성의 값이 객체라면 그 객체는 동결되지 않는다. 이는 constant에서 값의 재할당 방지를 위해서 문제가 있을 수 있게 되고, 이를 방지하는 방법을 사용하는게 좋다고 생각했다.
더 나아가서 deepFreeze 함수를 선언해서 사용하는 방법을 MDN에서 명시하고 있는데, 이 때에 객체 내부 참조 그래프를 그려볼때 순환을 포함하지 않는다는 것을 인지해야하고, 디자인을 기반으로 상황에 따라 패턴을 적용해야 하는 문제점이 발생한다. 

## Object.defineProperty( ... )
~~~javascript 
//constants.js
Object.defineProperty(exports, "PI", {
    value:        3.14,
    enumerable:   true,
    writable:     false,
    configurable: false
});
~~~
위의 방안으로 나온 디자인이다. 이 때에 객체를 생성할 때에 exports와 속성명을 인자로 받고([이거](https://nodejs.org/api/modules.html#modules_exports_shortcut)), exports 오브젝트의 속성으로 키와 밸류로 넣어주게 되는 것이다. 그런데 여기서 중요한 점은 writable: false와 configurable: false 로 둠으로 속성의 값들을 바꾸게 되는 것을 막는 것이다.([defineProberty](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)). 이렇게 만들어진 속성과 하위 객체는 값의 변경과 삭제 등이 불가능하게 된, constant와 같이 된다.
## function define( ... )
~~~javascript 
//constants.js
function define(name, value) {
    Object.defineProperty(exports, name, {
        value:      value,
        enumerable: true
    });
}

define("dbURL", "url.to.db.com");
//or like
define("naverCafe", {
	source:  "naverCafe",
	postSelectorData: {
		title:  "h3.title_text",
		articleUploadDate:  "div.article_info > span.date",
		articleAuthor:  "div.profile_info > div.nick_box > a.nickname",
		mainText:  "div.se-main-container, div.ContentRenderer",
		comment:  "span.text_comment",
		unnecessaryElements: []
	}
});
//useCase.js
let constants = require("constants.js");
console.log(constants.dbURL); // expect output: url.to.db.com
constants.dbURL = "aaaa";
console.log(constants.dbURL); // expect output: url.to.db.com

~~~
위의 방법은 뭐 한두개 만들면 좋은데, 각 값마다 다 writable, configurable, enumerable, 등등 설정해 주면 중복되는 코드가 너무 많아진다. 키와 밸류를 인자로 받아서 함수로 만들고 상수를 찍어내면 되겠다.


# 마무리
상수 선언에 관해서 어떤 방법이 있고 어떤 방법이 나은지에 대해서 좀 공부하다 보니 export와 require가 어떻게 동작하는지에 대해서 덤으로 공부하게 되었다. 대략적으로 ./이나 ../등으로 시작하지 않으면 node_modules에서 찾고, ./ 등이 붙으면 해당 경로에서 찾고, 그래도 없으면 LOAD_NODE_MODULES(X, dirname(Y))로 모듈을 찾아보는 것까지 하는건 알았는데, module.exports가 어떤 객체고 어떻게 동작 하는지는 모르고 있었는데 함께 공부하게 되어서 좋았다.

역시 삽질은 재밌다
