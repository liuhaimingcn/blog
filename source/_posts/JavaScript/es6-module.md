title: ES6-Module

date: 2016-2-2

tags:
    - es6
    - note

categories:
    - JavaScript
---

ES6的Class只是面向对象编程的语法糖，升级了ES5的构造函数的原型链继承的写法，并没有解决模块化问题。Module功能就是为了解决这个问题而提出的。  
在ES6之前，社区制定了一些模块加载方案，最主要的有CommonJS和AMD两种。前者用于服务器，后者用于浏览器。ES6在语言规格的层面上，实现了模块功能，而且实现得相当简单，完全可以取代现有的CommonJS和AMD规范，成为浏览器和服务器通用的模块解决方案。

    // CommonJS模块
    let { stat, exists, readFile } = require('fs');

上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取3个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

<!-- more --> 

    // ES6模块
    import { stat, exists, readFile } from 'fs';

上面代码的实质是从fs模块加载3个方法，其他方法不加载。这种加载称为“编译时加载”，即ES6可以在编译时就完成模块加载，效率要比CommonJS模块的加载方式高。当然，这也导致了没法引用ES6模块本身，因为它不是对象。

## 严格模式(ES5引入)

ES6的模块自动采用严格模式，不管你有没有在模块头部加上"use strict"。

## export命令 

模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。  
一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。

    // good
    export var firstName = 'Michael';
    export var lastName = 'Jackson';
    // best
    var firstName = 'Michael';
    var lastName = 'Jackson';
    export {firstName, lastName};

export命令除了输出变量，还可以输出函数或类（class）。

    function v1() { ... }
    function v2() { ... }
    export {
      v1 as streamV1,
      v2 as streamV2,
      v2 as streamLatestVersion
    };

export命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错，下面的import命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了ES6模块的设计初衷。  
export语句输出的值是动态绑定，绑定其所在的模块。

    export var foo = 'bar';
    setTimeout(() => foo = 'baz', 500);

上面代码输出变量foo，值为bar，500毫秒之后变成baz。

## import命令

使用export命令定义了模块的对外接口以后，其他JS文件就可以通过import命令加载这个模块（文件）。

    import {firstName, lastName, year} from './profile';

import命令接受一个对象（用大括号表示），里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同。  
如果想为输入的变量重新取一个名字，import命令要使用as关键字，将输入的变量重命名。

    import { lastName as surname } from './profile';

import命令具有提升效果，会提升到整个模块的头部，首先执行。

    foo();
    import { foo } from 'my_module';

如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起。

    export { es6 as default } from './someModule';
    // 等同于 (best)
    import { es6 } from './someModule';
    export default es6;

## 模块的整体加载

除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。

    import * as circle from './circle';
    console.log("圆面积：" \+ circle.area(4));
    console.log("圆周长：" \+ circle.circumference(14));

### export default命令

export default命令，为模块指定默认输出。一个模块只能有一个默认输出，因此export deault命令只能使用一次。

    export default function () {
      console.log('foo');
    }

其他模块加载该模块时，import命令可以为该匿名函数指定任意名字。这时import命令后面，不使用大括号。

    import customName from './export-default';
    customName(); // 'foo'

export default命令用在非匿名函数前，也是可以的。加载的时候，视同匿名函数加载。  
如果想在一条import语句中，同时输入默认方法和其他变量，可以写成下面这样。  

    import customName, { otherMethod } from './export-default';

## 模块的继承

    export * from 'circle';
    export var e = 2.71828182846;
    export default function(x) {
      return Math.exp(x);
    }

上面代码中的export *，表示再输出circle模块的所有属性和方法。注意，export *命令会忽略circle模块的default方法。然后，上面代码又输出了自定义的e变量和默认方法。

## ES6模块加载的实质
ES6模块加载的机制，与CommonJS模块完全不同。CommonJS模块输出的是一个值的拷贝，一旦输出一个值，模块内部的变化就影响不到这个值。而ES6模块输出的是值的引,用此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。。  
由于ES6输入的模块变量，只是一个”符号连接“，所以这个变量是只读的，对它进行重新赋值会报错。

## ES6模块的循环加载

ES6处理“循环加载”与CommonJS有本质的不同。ES6模块是动态引用，遇到模块加载命令import时，不会去执行模块，只是生成一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

## ES6模块的转码

浏览器目前还不支持ES6模块，为了现在就能使用，可以将转为ES5的写法。除了Babel可以用来转码之外，还有以下两个方法，也可以用来转码。

    ES6 module transpiler
    SystemJS

读阮一峰老师《ECMAScript6入门》一书的笔记    
<br>