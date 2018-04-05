title: setImmediate,setTimeout,nextTick的区别是什么

date: 2015-7-22

tags:
    - original

categories:
    - JavaScript
---
## 区别

1、nextTick和setImmediate主要的区别在于任务插入的位置nextTick的插入位置是在当前帧的末尾、io回调之前，如果nextTick过多，会导致io回调不断延后setImmediate的插入位置是在下一帧，不会影响io回调。

2、Nodejs的特点是事件驱动，异步I/O产生的高并发，产生此特点的引擎是事件循环，事件被分门别类地归到对应的事件观察者上，比如idle观察者，定时器观察者，I/O观察者等等，事件循环每次循环称为Tick，每次Tick按照先后顺序从事件观察者中取出事件进行处理。
调用setTimeout()时创建的计时器会被放入定时器观察者内部的红黑树中，每次Tick时，会从该红黑树中检查定时器是否超过定时时间，超过的话，就立即执行对应的回调函数。由于定时器是超时触发，这会导致触发精确度降低。

<!-- more -->  

## 代码例子

```
function nextTick(msg, cb) {
  process.nextTick(function() {
    console.log('tick: ' + msg);
    if (cb) {
      cb();
    }
  });
}

function immediate(msg, cb) {
  setImmediate(function() {
    console.log('immediate : ' + msg);
    if (cb) {
      cb();
    }
  });
}

nextTick('1');
nextTick('2', function() {
  nextTick('10');
});

immediate('3', function() {
  nextTick('5');
});

nextTick('7', function() {
  immediate('9');
});

immediate('4', function() {
  nextTick('8');
});

var n = 0;

const timer = setInterval(function() {
  n++;
  console.log('interval:', n);
  nextTick('tick from interval: ' +  n);
  nextTick('another tick from interval: ' +  n);
  immediate('immediate from interval: ' + n);
  immediate('another immediate from interval: ' + n);

  if ( n === 3 ) {
    clearInterval(timer);
  }
}, 0);

console.log('the last line of the program.');
```


## 结果

```
the last line of the program.
tick: 1
tick: 2
tick: 7
tick: 10
interval: 1
tick: tick from interval: 1
tick: another tick from interval: 1
immediate : 3
immediate : 4
immediate : 9
immediate : immediate from interval: 1
immediate : another immediate from interval: 1
tick: 5
tick: 8
interval: 2
tick: tick from interval: 2
tick: another tick from interval: 2
immediate : immediate from interval: 2
immediate : another immediate from interval: 2
interval: 3
tick: tick from interval: 3
tick: another tick from interval: 3
immediate : immediate from interval: 3
immediate : another immediate from interval: 3
```
## Maximum call stack size exceeded错误的原因

之所以会发生Maximum call stack size exceeded,因为process.maxTickDepth的缺省值是1000，如果递归调用nextTick只能调用1000次，超过1000就会报这个错，但并不是真正栈溢出，只是想给你一个提示不希望你递归调用nextTick太多次，如果nextTick递归调用，那么其他的回调事件就会等待，会造成event loop饥饿，所以官方推荐用setImmediate作为递归调用

## 资料来源

[求科普setImmediate API](https://cnodejs.org/topic/519b523c63e9f8a5429b25e3)  
[setTimeout，setInterval，process.nextTick，setImmediate in Nodejs](http://www.cnblogs.com/kongxianghai/p/3942226.html)

<br>
