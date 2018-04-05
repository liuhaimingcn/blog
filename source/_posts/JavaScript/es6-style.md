title: ES6-Module

date: 2016-2-2

tags:
    - es6
    - note

categories:
    - JavaScript
---

# 规格文件

规格文件是计算机语言的官方标准，详细描述语法规则和实现方法。  
ECMAScript 6的规格，可以在ECMA国际标准组织的官方网站（www.ecma-international.org/ecma-262/6.0/）免费下载和在线阅读。

<!-- more --> 

# 编程风格

## 块级作用域

let取代var  
全局常量和线程安全
在let和const之间，建议优先使用const，尤其是在全局环境，不应该设置变量，只应设置常量。这符合函数式编程思想，有利于将来的分布式运算。

    // bad
    var a = 1, b = 2, c = 3;
    // good
    const a = 1;
    const b = 2;
    const c = 3;

const声明常量还有两个好处，一是阅读代码的人立刻会意识到不应该修改这个值，二是防止了无意间修改变量值所导致的错误。  
所有的函数都应该设置为常量。

## 严格模式

V8引擎只在严格模式之下，支持let。  

## 字符串

静态字符串一律使用单引号或反引号，不使用双引号。动态字符串使用反引号。

    // bad
    const a = "foobar";
    const b = 'foo' + a + 'bar';
    // good
    const a = 'foobar';
    const b = `foo${a}bar`;

## 解构赋值

使用数组成员对变量赋值时，优先使用解构赋值。

    const arr = [1, 2, 3, 4];
    // bad
    const first = arr[0];
    const second = arr[1];
    // good
    const [first, second] = arr;

函数的参数如果是对象的成员，优先使用解构赋值。

    // bad
    function getFullName(user) {
      const firstName = user.firstName;
      const lastName = user.lastName;
    }
    // best
    function getFullName({ firstName, lastName }) {
    }

如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。这样便于以后添加返回值，以及更改返回值的顺序。

    // bad
    function processInput(input) {
      return [left, right, top, bottom];
    }
    // good
    function processInput(input) {
      return { left, right, top, bottom };
    }
    const { left, right } = processInput(input);

## 数组

使用扩展运算符（...）拷贝数组。

    // bad
    const len = items.length;
    const itemsCopy = [];
    for (let i = 0; i < len; i++) {
      itemsCopy[i] = items[i];
    }
    // good
    const itemsCopy = [...items];

使用Array.from方法，将类似数组的对象转为数组。

    const foo = document.querySelectorAll('.foo');
    const nodes = Array.from(foo);

立即执行函数可以写成箭头函数的形式。

    (() => {
      console.log('Welcome to the Internet.');
    })();

那些需要使用函数表达式的场合，尽量用箭头函数代替。因为这样更简洁，而且绑定了this。

    // bad
    [1, 2, 3].map(function (x) {
      return x * x;
    });
    // best
    [1, 2, 3].map(x => x * x);

简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体较为复杂，行数较多，还是应该采用传统的函数写法。  

不要在函数体内使用arguments变量，使用rest运算符（...）代替。因为rest运算符显式表明你想要获取参数，而且arguments是一个类似数组的对象，而rest运算符可以提供一个真正的数组。

    // bad
    function concatenateAll() {
      const args = Array.prototype.slice.call(arguments);
      return args.join('');
    }
    // good
    function concatenateAll(...args) {
      return args.join('');
    }

使用默认值语法设置函数参数的默认值。

    // bad
    function handleThings(opts) {
      opts = opts || {};
    }
    // good
    function handleThings(opts = {}) {
      // ...
    }

所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数。

    // bad
    function divide(a, b, option = false ) {
    }
    // good
    function divide(a, b, { option = false } = {}) {
    }

## Map结构

注意区分Object和Map，只有模拟实体对象时，才使用Object。如果只是需要key: value的数据结构，使用Map结构。因为Map有内建的遍历机制。

    let map = new Map(arr);
    for (let key of map.keys()) {
      console.log(key);
    }
    for (let value of map.values()) {
      console.log(value);
    }
    for (let item of map.entries()) {
      console.log(item[0], item[1]);
    }

## Class

总是用Class，取代需要prototype的操作。因为Class的写法更简洁，更易于理解。

    // bad
    function Queue(contents = []) {
      this._queue = [...contents];
    }
    Queue.prototype.pop = function() {
      const value = this._queue[0];
      this._queue.splice(0, 1);
      return value;
    }
    // good
    class Queue {
      constructor(contents = []) {
        this._queue = [...contents];
      }
      pop() {
        const value = this._queue[0];
        this._queue.splice(0, 1);
        return value;
      }
    }

使用extends实现继承，因为这样更简单，不会有破坏instanceof运算的危险。

    // bad
    const inherits = require('inherits');
    function PeekableQueue(contents) {
      Queue.apply(this, contents);
    }
    inherits(PeekableQueue, Queue);
    PeekableQueue.prototype.peek = function() {
      return this._queue[0];
    }
    // good
    class PeekableQueue extends Queue {
      peek() {
        return this._queue[0];
      }
    }

## 模块

Module语法是JavaScript模块的标准写法，坚持使用这种写法。使用import取代require。

    // bad
    const moduleA = require('moduleA');
    const func1 = moduleA.func1;
    const func2 = moduleA.func2;
    // good
    import { func1, func2 } from 'moduleA';

使用export取代module.exports。  
如果模块只有一个输出值，就使用export default，如果模块有多个输出值，就不使用export default，不要export default与普通的export同时使用。

    // commonJS的写法
    var React = require('react');
    var Breadcrumbs = React.createClass({
      render() {
        return <nav />;
      }
    });
    module.exports = Breadcrumbs;
    // ES6的写法
    import React from 'react';
    const Breadcrumbs = React.createClass({
      render() {
        return <nav />;
      }
    });
    export default Breadcrumbs

如果模块默认输出一个函数，函数名的首字母应该小写。

    function makeStyleGuide() {
    }
    export default makeStyleGuide;

如果模块默认输出一个对象，对象名的首字母应该大写。

    const StyleGuide = {
      es6: {
      }
    };
    export default StyleGuide;

读阮一峰老师《ECMAScript6入门》一书的笔记    
<br>