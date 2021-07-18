---
layout: post
title: "MVC, MVP, MVVM"
subtitle: "디자인 패턴과 그 의미"
date: 2021-07-17 12:30:15 +0900
background: '/img/posts/05.jpg'
---


# MVC, MVP, MVVC???
일단 이 세 가지에 대해서 설명하기 전에, 이 패턴들이 왜 나왔는지 부터 짚고 넘어가자.
디자인 패턴이란 작업 공간에서 각 계층을 분리시킴으로 코드의 재활용성을 높이고 불필요한 중복을 막기 위해서 도입되었다. 이때 가장 많이 사용되는 패턴이 이 MVC, MVP, MVVM 패턴이고 이는 각 
1. MVC => Model - View - Controller
2. MVP => Model - View - Presenter
3. MVVM => Model = View = ViewModel
로 구성된다. 

## MVC
개발자가 Controller에 로직을 다루고, Controller는 Model을 통해 가져온 데이터를 View에게 전달한다.
여기서 View는 웹 레이어, Controller는 서버, Model은 데이터베이터베이스라고 할 수 있다.
결국 View에서는 사용자가 보는 모든 것들이 될 것이고, Controller는 사용자가 보는 뒷편 데이터베이스 앞편의 모든 것들이 될것이고, Model은 저장되는 모든 데이터들이 되는 것이다. 

또 MVC에서의 COntroller는 Mdoel에게 직접적인 영향을 끼칠 수 었지만, Model은 Controller와 VIew에게 직접적인 관련은 전혀 없고, 간접적인 방식으로 전달하게 된다. 

이제 Controller는 View에게 영향을 끼치지만 그 반대는 성립되지 않는다.
Controller는 원하는 View를 선택하여 작용할 수 있지만 View가 Controller를 선택하여 행동할 수는 없다는 것이다.
결국 View와 Controller의 관계는 1:N 관계이며, View는 Controller에 영향을 끼칠 수 없는 관계가 된다.

그리고, MVP와 MVVM은 이 MVC 패턴에서 파생되어 나온 것이 되겠다.

![설명](/img/posts/21_07_18_MVC/MVC.png "도식도")

## MVP
MVP가 MVC에서 파생되어 나온 패턴인 만큼, 유사한 부분들이 존재 한다.
Controller의 자리에 Presenter가 들어간 것 외에는 거진 다 유사한 부분이라고생각 할 수 있다. 
데이터가 처리되는 순서를 살펴 보자면
1. 사용자의 입력이 View를 통해서 들어온다.
2. View는 해당 데이터를 Presenter에게 요청한다.
3. Presenter는 Model한테 데이터를 요청한다
4. Model은 Presenter에서 요청받은 데이터를 전달한다
5. Presenter는 View에게 데이터를 전달한다
6. View는 Prsenter가 응답한 데이터를 이용하여 화면에 표시한다


이와 같은 방법으로 기능할 때, View 와 Presenter는 1:1 관계가 된다.
View는 Presenter를 호출하고 presenter는 Model을 호출한다. 

이와 같은 패턴 사용 시에 대부분의 책임을 Presenter와 Model이 가지고 있어 View는 출력만 하는 역할을 하게 되고, 테스트 시 View에 대한 책임이 분리되어 있기 때문에 각 요소들을 독립적으로 테스팅 할 수 있게 되며 View와 Model이 의존하는 관계를 없앨 수 있게 되었다.
하지만 단점으로 View와 presenter가 1:1 관계이기 때문에 코드의 양이 늘어나게 되고, View와 Presenter 간의 높은 의존성이라는 문제가 발생하게 된다.
![설명](/img/posts/21_07_18_MVC/MVP.png "도식도")

## MVVM 
이 또한도 MVC패턴과 비슷하게, Model과 View가 존재한다. 하지만 Controller 대신에 ViewModel이라는 것이 존재하게 되고, 이 ViewModel은 View를 표현하기 위해서 존재하는 Model이다. 또 View를 나타내기 위한 데이터를 처리하는 역할을 지니게 된다

데이터가 처리되는 순서를 볼 때에, 
1. 사용자의 입력값이 View를 통해 들어온다
2. View에 입력값이 들어오면 ViewModel에 입력값을 전달한다. 그 후 ViewModel은 Model에게 데이터 요청을 보낸다.
3. Model은 ViewModel에게 요청받은 데이터를 응답하고 ViewModel은 그 값을 받은 후 가공처리하여 내부에 저장한다.
4. View는 ViewModel과 Data Binding통해 화면상에 표출한다.

위와 같은 순서에서, Data Binding이라는 기능을 사용하게 됨으로 View와 ViewModel간의 의존성이 없게 된다. 의존성이 없기 때문에 모듈화가 가능하게 되고, 의존성을 Data Binding과 Event와 Notify로서 해결한다.


![설명](/img/posts/21_07_18_MVC/MVVM.png "도식도")


Vue.js는 이 MVVM 패턴을 기반으로 형성되었다. Vue.js에서 예제 코드 하나를 살펴보자면,

``` html
<html>
    <head>
        <meta charset="UTF-8"/>
        <title>title here</title>
    </head>
    <!--  View Start-->
    <body>
        <div id="app">
            {{message}}   <!--  Directive to present Data-->
        </div>
    </body>
    <!--  View End-->
    <script src="https://unpkg.com/vue" ></script>
    <script>
        // Model Start
        let modle={
            message: "hi vue.js!",
        };
        // Model End

        // ViewModel Start (Vue Object)
        let vueInstance = new Vue({
            el: "#app",
            data: model,
        });
        // ViewModel End
    </script>
</html>

```
위의 예제에서 Vue에서 Model, ViewModel, View가 어떻게 엮여져 있는지를 잘 보여주고 있다. Model의 데이터는 ViewModel을 통헤서 View의 Directive, 즉 명령을 통해서 렌더링되어서 유저에게 보여지게 된다.

MVVM은 이정도 예제로 충분하지 않을까..?


# 결론
MVC, MVP, MVVM의 각 예제들이 있지만 주로 자주 사용되는 분야들이 조금씩 틀린 것 같다. MVC는 주로 Spring으로 개발하는 쪽에서 자주 나오고, MVP는 안드로이드 개발, MVVM은 Vue 개발 사이트에서 자주 보였다. Spring, 안드로이드, Vue 개발을 하다가 MVC MVP MVVM이 나온건지 패턴의 개념적 정의가 나온 후에 각 진영에 이게 맞는것 같다 하여 패턴을 도입하기 시작한지는 알 수 없지만, 예의 패턴들이 자주 쓰이고 있고 도입할 때의 장점이 명확하기 때문에 미리미리 공부 해놓는게 좋아보인다. 

또 찾아보니까 MVC가 발전한 형태은 MVC2도 있다고 하는..데...
이건 또 언제 공부한데...