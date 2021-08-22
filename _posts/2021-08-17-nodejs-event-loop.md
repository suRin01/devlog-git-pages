---
layout: post
title: "Nodejs의 이벤트 루프"
subtitle: "이벤트 루프 개념잡기"
date: 2021-08-18 12:30:15 +0900
background: '/img/posts/05.jpg'
---





# 

microtask queue(job queue)           > animation frames >         task queue(event queue)
promise.then/catch, process.nextTick                                      timer
promise 내부까지는 동기(resolve 등)
.then을 만나는 순간 비동기