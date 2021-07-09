---
layout: post
title: "Nodejs Body parser?"
subtitle: "Body-Parser의 변경점"
date: 2021-07-08 12:30:15 +0900
background: '/img/posts/05.jpg'
---


# Express.js 에서 Body-Parser의 변경사항

이전에 따로 npm을 통해서 설치해 주어야 했던 Body-Parser가 express.js 속으로 포함되었다.[링크](https://expressjs.com/en/4x/api.html#express-json-middleware)

그래서 기존에 사용했던
``` javascript
// Express.js@<4.16.0
const express = require("express");
const app = express();
const bodyParser = require('body-parser');
app.use(bodyParser());

app.post("/postuser", (req, res) => {
    const name = req.body.name;
    const age = req.body.age;
    console.log(name, age);  // postmanTest1 10

    res.send("Hello postuser!");
});

```

의 코드가, 다음과 같이 변경되었다.

``` javascript
// Express.js@>=4.16.0
const express = require("express");
const app = express();
app.use(express.json());

app.post("/postuser", (req, res) => {
    const name = req.body.name;
    const age = req.body.age;
    console.log(name, age); // postmanTest1 10

    res.send("Hello postuser!");
});

```




# 실제 테스트

## Express.js 4.16.0 이후
라우팅 스크립트
``` javascript
...
router.post("/postuser", (req, res) => {
    console.log(req.body);
    const name = req.body.name;
    const age = req.body.age;

    userService.insertUserList(name, age);

    console.log(userService.getUserList());
    res.send("Hello postuser!");
});
...
```

postman 통한 테스트
![신규포스트맨](/img/posts/21_07_08/express-new-postman.png "2")

![신규콘솔](/img/posts/21_07_08/express-new-console.png "3")


## Express.js 4.16.0 이전
``` javascript 

const express = require('express');
const app = express();

const bodyParser = require('body-parser');
app.use(bodyParser());

app.post("/postuser", (req, res) => {
    const name = req.body.name;
    const age = req.body.age;
    console.log(name, age);

    res.send("Hello postuser!");
});

 
app.listen(4000)
```

![클래스](/img/posts/21_07_08/express-old-postman.png "5")

![클래스](/img/posts/21_07_08/express-old-console.png "6")