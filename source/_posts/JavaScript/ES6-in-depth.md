title: 深入浅出ES6笔记  
date: 2016-2-18  
tags:
    - es6
    - note
categories:
    - JavaScript
---

# ES6 是什么

* 编程语言 JavaScript 是 ECMAScript 的实现和扩展,由 ECMA(一个类似 W3C 的标准组织)参与进行标准化。   
* ES4 饱受争议,当标准委员会最终停止开发 ES4 时,其成员同意发布一个相对谦和的 ES5 版本,随后继续制定一些更具实质性的新特性。这一明确的协商协议最终命 名为“Harmony”(ES6)。  
* 2009 年发布的改进版本 ES5,引入了 Object.create()、Object.defineProperty()、getters 和 setters、严格模式以及 JSON 对象。  
* 如果你想在 web 环境中使用这种新语法,同时需要支持 IE 和 Safari, 你可以使用 Babel 或 Google 的 Traceur 这些编译器来将你的 ES6 代码翻译为 Web 友好 的 ES5 代码。  

<!-- more -->

# 迭代器和 for-of 循环
## 遍历数组中的元素

* 20 年前 JavaScript 刚萌生时     

        for (var index = 0; index < myArray.length; index++) { 
            console.log(myArray[index]);
        }

* ES5 正式发布后  

        myArray.forEach(function (value) { 
            console.log(value);
        });  

    这段代码看起来更加简洁,但这种方法也有一个小缺陷:你不能使用 break 语句中 断循环,也不能使用 return 语句返回到外层函数。  

* for-in 循环(千万别这样做, 是为普通对象设计的) 

        for (var index in myArray) {
            console.log(myArray[index]);
        }

    在这段代码中,赋给index的值不是实际的数字,而是字符串“0”、“1”、“2”, 此时很可能在无意之间进行字符串算数计算,例如:“2” + 1 == “21”,这给 编码过程带来极大的不便。  
    作用于数组的 for-in 循环体除了遍历数组元素外,还会遍历自定义属性。举个例子,如果你的数组中有一个可枚举属性 myArray.name,循环将额外执行 一次,遍历到名为“name”的索引。就连数组原型链上的属性都能被访问到。  
    最让人震惊的是,在某些情况下,这段代码可能按照随机顺序遍历数组元素。  
    简而言之,for-in 是为普通对象设计的,你可以遍历得到字符串类型的键,因此不适用于数组遍历。

* ES6

        for (var value of myArray) {
            console.log(value);
        }

    for-in 循环用来遍历对象属性。  
    for-of 循环用来遍历数据—例如数组中的值。

## for-of 循环

    for (var value of myArray) {
      console.log(value);
    }

这是最简洁、最直接的遍历数组元素的语法  
这个方法避开了 for-in 循环的所有缺陷  
与 forEach()不同的是,它可以正确响应 break、continue 和 return 语句  
for-of 循环不仅支持数组,还支持大多数类数组对象,例如 DOM NodeList 对象。  
for-of 循环也支持字符串遍历,它将字符串视为一系列的 Unicode 字符来进行遍历  

    var str = 'liuhaiming';
    for(var chr of str) {
      console.log(chr);
    }

for-of 循环也支持Set 对象  

    var uniqueWords = new Set(words);
    for (var word of uniqueWords) {
      console.log(word);
    }

for-of 循环也支持Map对象  

    for (var [key, value] of phoneBookMap) {
      console.log(key + "'s phone number is: " + value);
    }

for-of 循环不支持普通对象,但如果你想迭代一个对象的属性,你可以用 for-in 循 环(这也是它的本职工作)或内建的 Object.keys()方法:

    // 向控制台输出对象的可枚举属性
    for (var key of Object.keys(someObject)) {
      console.log(key + ": " + someObject[key]);
    }



<br>