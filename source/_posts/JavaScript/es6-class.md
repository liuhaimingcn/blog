title: ES6-Class基本语法

date: 2016-2-2

tags:
    - es6
    - note

categories:
    - JavaScript
---

## Class基本语法 

ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。
​	
    // old
      function Point(x,y){
      this.x = x;
      this.y = y;
    }
    Point.prototype.toString = function () {
      return '(' + this.x + ', ' + this.y + ')';
    }
    // new
    class Point {
      constructor(x, y) {
        this.x = x;
        this.y = y;
      }
      toString() {
        return '(' + this.x + ', ' + this.y + ')';
      }   
    }

<!-- more --> 

Point类除了构造方法，还定义了一个toString方法。注意，定义“类”的方法的时候，前面不需要加上function这个保留字，直接把函数定义放进去了就可以了。另外，方法之间不需要逗号分隔，加了会报错。  
构造函数的prototype属性，在ES6的“类”上面继续存在。事实上，类的所有方法都定义在类的prototype属性上面。  
类的内部所有定义的方法，都是不可枚举的（enumerable）。这一点与ES5的行为不一致。

    class Point {
      constructor(x, y) {}
      toString() {}
    }
    Object.keys(Point.prototype) // []
    Object.getOwnPropertyNames(Point.prototype) // ["constructor","toString"]

constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加。  
constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象。

    class Foo {
      constructor() {
        return Object.create(null);
      }
    }
    new Foo() instanceof Foo; // false 

生成实例对象的写法，与ES5完全一样，也是使用new命令。如果忘记加上new，像函数那样调用Class，将会报错。  
与ES5一样，实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）  
与ES5一样，类的所有实例共享一个原型对象。所以__proto__属性是相等的。  
name属性总是返回紧跟在class关键字后面的类名。

    class Point {}
    Point.name // "Point"

下面代码使用表达式定义了一个类。需要注意的是，这个类的名字是MyClass而不是Me，Me只在Class的内部代码可用，指代当前类。

    const MyClass = class Me {
      getClassName() {
        return Me.name;
      }
    };

如果Class内部没用到的话，可以省略Me。  
采用Class表达式，可以写出立即执行的Class。  
Class不存在变量提升（hoist），这一点与ES5完全不同。

    new Foo(); // ReferenceError
    class Foo {}

类和模块的内部，默认就是严格模式，所以不需要使用use strict指定运行模式。  
考虑到未来所有的代码，其实都是运行在模块之中，所以ES6实际上把整个语言升级到了严格模式。

## Class的继承

Class之间可以通过extends关键字实现继承，这比ES5的通过修改原型链实现继承，要清晰和方便很多。

    class ColorPoint extends Point {}

下面代码中，constructor方法和toString方法之中，都出现了super关键字，它指代父类的实例（即父类的this对象）。

    class ColorPoint extends Point {
      constructor(x, y, color) {
        super(x, y); // 调用父类的constructor(x, y)
        this.color = color;
      }
      toString() {
        return this.color + ' ' + super.toString(); // 调用父类的toString()
      }
    }

子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。  
子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型。

## 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。
ES6允许继承原生构造函数定义子类，因为ES6是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。

    class MyArray extends Array {
      constructor(...args) {
        super(...args);
      }
    }
    var arr = new MyArray();
    arr[0] = 12;
    arr.length // 1
    arr.length = 0;
    arr[0] // undefined

## Class的取值函数（getter）和存值函数（setter）  

与ES5一样，在Class内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

    class MyClass {
      constructor() {
        // ...
      }
      get prop() {
        return 'getter';
      }
      set prop(value) {
        console.log('setter: '+value);
      }
    }
    let inst = new MyClass();
    inst.prop = 123; // setter: 123
    inst.prop // 'getter'

## Class的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

    class Foo {
      static classMethod() {
        return 'hello';
      }
    }
    Foo.classMethod() // 'hello'
    var foo = new Foo();
    foo.classMethod() // TypeError: undefined is not a function

父类的静态方法，可以被子类继承。  
静态方法也是可以从super对象上调用的。

## Class的静态属性

静态属性指的是Class本身的属性，即Class.propname，而不是定义在实例对象（this）上的属性。

    class Foo {
    }
    Foo.prop = 1;
    Foo.prop // 1

目前，只有这种写法可行，因为ES6明确规定，Class内部只有静态方法，没有静态属性。

## new.target属性

new是从构造函数生成实例的命令。ES6为new命令引入了一个new.target属性，（在构造函数中）返回new命令作用于的那个构造函数。如果构造函数不是通过new命令调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

    function Person(name) {
      if (new.target !== undefined) {
        this.name = name;
      } else {
        throw new Error('必须使用new生成实例');
      }
    }
    // 另一种写法
    function Person(name) {
      if (new.target === Person) {
        this.name = name;
      } else {
        throw new Error('必须使用new生成实例');
      }
    }
    var person = new Person('张三'); // 正确
    var notAPerson = Person.call(person, '张三'); // 报错

Class内部调用new.target，返回当前Class。  
子类继承父类时，new.target会返回子类。

    class Rectangle {
      constructor(length, width) {
        console.log(new.target === Rectangle);
        // ...
      }
    }
    class Square extends Rectangle {
      constructor(length) {
        super(length, length);
      }
    }
    var obj = new Square(3); // 输出 false

利用这个特点，可以写出不能独立使用、必须继承后才能使用的类。

## 类的修饰

修饰器（Decorator）是一个表达式，用来修改类的行为。这是ES7的一个提案，目前Babel转码器已经支持。  
修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。  
​    
读阮一峰老师《ECMAScript6入门》一书的笔记    
<br>