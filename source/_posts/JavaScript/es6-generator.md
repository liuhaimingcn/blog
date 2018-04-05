title: ES6-Generator函数

date: 2015-10-28

tags:
    - es6
    - note

categories:
    - JavaScript
---

## 基本概念
Generator函数是ES6提供的一种异步编程解决方案  
Generator函数有两个特征。一是，function命令与函数名之间有一个星号；二是，函数体内部使用yield语句，定义不同的内部状态。  
调用Generator函数后，该函数并不执行，返回的是一个指向内部状态的指针对象（遍历器对象）。下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield语句（或return语句）为止。  

    function* helloWorldGenerator() {
      yield 'hello';
      yield 'world';
      return 'ending';
    }
    var hw = helloWorldGenerator();
    hw.next() // { value: 'hello', done: false }
    hw.next() // { value: 'world', done: false }
    hw.next() // { value: 'ending', done: true }
    hw.next() // { value: undefined, done: true }

每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield语句后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

<!-- more --> 

##yield语句
遍历器对象的next方法的运行逻辑如下。

    遇到yield语句，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。
    下一次调用next方法时，再继续往下执行，直到遇到下一个yield语句。
    如果没有再遇到新的yield语句，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。
    如果该函数没有return语句，则返回的对象的value属性值为undefined。

yield语句后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为JavaScript提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。  
一个函数里面，只能执行一次（或者说一个）return语句，但是可以执行多次（或者说多个）yield语句。  
yield语句不能用在普通函数中，否则会报错。  
yield语句如果用在一个表达式之中，必须放在圆括号里面。yield语句用作函数参数或赋值表达式的右边，可以不加括号。

##next方法的参数
yield句本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield语句的返回值。  
由于next方法的参数表示上一个yield语句的返回值，所以第一次使用next方法时，不能带有参数。V8引擎直接忽略第一次使用next方法时的参数，只有从第二次使用next方法开始，参数才是有效的。

##for...of循环
for...of循环可以自动遍历Generator函数，且此时不再需要调用next方法。这里需要注意，一旦next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象。  
for...of循环、扩展运算符（...）、解构赋值和Array.from方法内部调用的，都是遍历器接口。这意味着，它们可以将Generator函数返回的Iterator对象，作为参数。

    function* numbers () {
      yield 1
      yield 2
      return 3
      yield 4
    }
    [...numbers()] // [1, 2]
    Array.from(numbers()) // [1, 2]
    let [x, y] = numbers();
    x // 1
    y // 2
    for (let n of numbers()) {
      console.log(n)
    }
    // 1
    // 2

##Generator.prototype.throw()
Generator函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在Generator函数体内捕获。  
如果Generator函数内部没有部署try...catch代码块，那么throw方法抛出的错误，将被外部try...catch代码块捕获。  
如果Generator函数内部部署了try...catch代码块，那么遍历器的throw方法抛出的错误，不影响下一次遍历，否则遍历直接终止。  
Generator函数内抛出的错误，也可以被函数体外的catch捕获。

##Generator.prototype.return()
Generator函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数。

    function* gen() {
      yield 1;
      yield 2;
      yield 3;
    }
    var g = gen();
    g.next()        // { value: 1, done: false }
    g.return("foo") // { value: "foo", done: true }
    g.next()        // { value: undefined, done: true }

如果return方法调用时，不提供参数，则返回值的vaule属性为undefined。  
如果Generator函数内部有try...finally代码块，那么return方法会推迟到finally代码块执行完再执行。

## yield*语句
yield*语句，用来在一个Generator函数里面执行另一个Generator函数。  
yield*语句等同于在Generator函数内部，部署一个for...of循环。  
如果yield*后面跟着一个数组，由于数组原生支持遍历器，因此就会遍历数组成员。

    function* gen(){    
      yield* ["a", "b", "c"];
    }
    gen().next() // { value:"a", done:false }

##作为对象属性的Generator函数
```
let obj = {
  * myGeneratorMethod() {
    ···
  }
}; 
```

等价于

```
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```

##构造函数是Generator函数 // ？

##Generator函数推导 // ?

##Generator与状态机 // ?

##Generator与协程 // ?

##异步操作的同步化表达
Generator函数的暂停执行的效果，意味着可以把异步操作写在yield语句里面，等到调用next方法时再往后执行。

```
  function* main() {
    var result = yield request("http://some.url");
    var resp = JSON.parse(result);
      console.log(resp.value);
  }
  function request(url) {
    makeAjaxCall(url, function(response){
      it.next(response);
    });
  }
  var it = main();
  it.next();
```

##控制流管理 // ???

##部署iterator接口 // ?

##作为数据结构 // ?

读阮一峰老师《ECMAScript6入门》一书的笔记    
<br>