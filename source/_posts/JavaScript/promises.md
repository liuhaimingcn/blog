title: We have a problem with promises  
date: 2016-2-23  
tags:
    - note
categories:
    - JavaScript
---

## 下面的四种 promises 的区别是什么

``` js  
doSomething().then(function () {  
    return doSomethingElse();  
});  
  
doSomething().then(function () {  
    doSomethingElse();  
});  
  
doSomething().then(doSomethingElse());  

doSomething().then(doSomethingElse);
```

<!-- more -->

### 原文地址链接：http://fex.baidu.com/blog/2015/07/we-have-a-problem-with-promises/

Puzzle #1

``` js
doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|
```

Puzzle #2

``` js
doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                  finalHandler(undefined)
                  |------------------|
```

Puzzle #3

``` js
doSomething
|-----------------|
doSomethingElse(undefined)
|---------------------------------|
                  finalHandler(resultOfDoSomething)
                  |------------------|
```

Puzzle #4

``` js
doSomething
|-----------------|
                  doSomethingElse(resultOfDoSomething)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|
```

Promises 解决了 Callback Hell 问题，并且不仅仅是缩进问题。就像在 《Callback Hell 的救赎》 中描述的一样，回调函数真正的问题在于他剥夺了我们使用 return 和 throw 这些关键字的能力。

## Promises的正确的风格

``` js
remotedb.allDocs(...).then(function (resultOfAllDocs) {
  return localdb.put(...);
}).then(function (resultOfPut) {
  return localdb.get(...);
}).then(function (resultOfGet) {
  return localdb.put(...);
}).catch(function (err) {
  console.log(err);
}); 
```

这种写法被称为 composing promises ，是 promises 的强大能力之一。每一个函数只会在前一个 promise 被调用并且完成回调后调用，并且这个函数会被前一个 promise 的输出调用

## 用了 promises 后怎么用 forEach?

``` js
// 错误写法
// I want to remove() all docs
db.allDocs({include_docs: true}).then(function (result) {
  result.rows.forEach(function (row) {
    db.remove(row.doc);  
  });
}).then(function () {
  // I naively believe all docs have been removed() now!
});
// 正确写法
db.allDocs({include_docs: true}).then(function (result) {
  return Promise.all(result.rows.map(function (row) {
    return db.remove(row.doc);
  }));
}).then(function (arrayOfResults) {
  // All docs have really been removed() now!
});
```

一定不要忘记使用 .catch()

## catch() 与 then(null, ...) 并非完全等价

catch() 仅仅是一个语法糖。因此下面两段代码是等价的:

``` js
somePromise().catch(function (err) {
  // handle error
});
somePromise().then(null, function (err) {
  // handle error
});
```

但是并非完全等价

``` js
somePromise().then(function () {
  throw new Error('oh noes');
}).catch(function (err) {
  // I caught your error! :)
});

somePromise().then(function () {
  throw new Error('oh noes');
}, function (err) {
  // I didn't catch your error! :(
});
```


<br>