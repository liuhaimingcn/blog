title: ES6-字符串的扩展

date: 2015-10-26

tags:
    - es6
    - note

categories:
    - JavaScript
---

##字符的Unicode表示法

   之前不识别在“\u”后面跟上超过0xFFFF的数值（比如\u20BB7），现在用只要将码点放入大括号中就能识别了（比如\u{20BB7}）  
   所以现在JavaScript共有6种方法可以表示一个字符
​        
        '\z' === 'z'  // true  
        '\172' === 'z' // true  
        '\x7A' === 'z' // true  
        '\u007A' === 'z' // true  
        '\u{7A}' === 'z' // true        

<!-- more -->      

##codePointAt() // ?

   JavaScript内部，字符以UTF-16的格式储存，每个字符固定为2个字节。对于那些需要4个字节储存的字符（Unicode码点大于0xFFFF的字符），JavaScript会认为它们是两个字符。

        var s = "𠮷";
        s.length // 2
        s.charAt(0) // ''
        s.charAt(1) // ''
        s.charCodeAt(0) // 55362
        s.charCodeAt(1) // 57271

   ES6提供了codePointAt方法，能够正确处理4个字节储存的字符，返回一个字符的码点。

##String.fromCodePoint()

   ES5提供String.fromCharCode方法，用于从码点返回对应字符，但是这个方法不能识别辅助平面的字符（编号大于0xFFFF）。

        String.fromCharCode(0x20BB7) // "ஷ"

   ES6提供了String.fromCodePoint方法，可以识别0xFFFF的字符，弥补了String.fromCharCode方法的不足。

        String.fromCodePoint(0x20BB7) // "𠮷"

##字符串的遍历器接口

   ES6为字符串添加了遍历器接口，使得字符串可以被for...of循环遍历。  
   除了遍历字符串，这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点。

##at() // ES7

   ES5对字符串对象提供charAt方法，返回字符串给定位置的字符。该方法不能识别码点大于0xFFFF的字符。  
   ES7提供了字符串实例的at方法，可以识别Unicode编号大于0xFFFF的字符，返回正确的字符。

##normalize() // ?

   ES6提供String.prototype.normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为Unicode正规化。

##includes(), startsWith(), endsWith()
   传统上，JavaScript只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6又提供了三种新方法。  

        includes()：返回布尔值，表示是否找到了参数字符串。
        startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部。
        endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部。

   这三个方法都支持第二个参数，endsWith的第二个参数表示它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束。

        var s = 'Hello world!';
        s.startsWith('world', 6) // true
        s.endsWith('Hello', 5) // true
        s.includes('Hello', 6) // false

##repeat()

   repeat方法返回一个新字符串，表示将原字符串重复n次。

        'hello'.repeat(2) // "hellohello"
        'na'.repeat(0) // ""

   参数如果是小数，会被取整。  
   如果repeat的参数是负数或者Infinity，会报错。  
   但是，如果参数是0到-1之间的小数，则等同于0，这是因为会先进行取整运算。0到-1之间的小数，取整以后等于-0，repeat视同为0。 
   参数NaN等同于0。  
   如果repeat的参数是字符串，则会先转换成数字。

##模板字符串

   模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。  
   如果在模板字符串中需要使用反引号，则前面要用反斜杠转义。  
   如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。  
   模板字符串中嵌入变量，需要将变量名写在${}之中。  
   大括号内部可以放入任意的JavaScript表达式，可以进行运算，以及引用对象属性。  
   模板字符串之中还能调用函数。  
   如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的toString方法。  
   如果模板字符串中的变量没有声明，将报错。  
   由于模板字符串的大括号内部，就是执行JavaScript代码，因此如果大括号内部是一个字符串，将会原样输出。

##标签模板

   模板字符串可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。  
   tag函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，其他参数，都是模板字符串各个变量被替换后的值。
​      
        var a = 5;
        var b = 10;
        function tag(s, v1, v2) {
          console.log(s[0]); // "Hello "
          console.log(s[1]); // " world "
          console.log(s[2]); // ""
          console.log(v1); // 15
          console.log(v2); // 50
          return "OK";
        }
        tag`Hello ${ a + b } world ${ a * b}`; // "OK"  等同于 tag(['Hello ', ' world ', ''], 15, 50)

   “标签模板”的一个重要应用，就是过滤HTML字符串，防止用户输入恶意内容。

        var message = SaferHTML`<p>${sender} has sent you a message.</p>`;
        function SaferHTML(templateData) {
          var s = templateData[0];
          for (var i = 1; i < arguments.length; i++) {
            var arg = String(arguments[i]);
            // Escape special characters in the substitution.
            s += arg.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
            // Don't escape special characters in the template.
            s += templateData[i];
          }
          return s;
        }

   模板处理函数的第一个参数（模板字符串数组），还有一个raw属性。该数组的成员与strings数组完全一致。两者唯一的区别，就是字符串里面的斜杠都被转义了。比如，strings.raw数组会将\n视为\和n两个字符，而不是换行符。这是为了方便取得转义之前的原始模板而设计的。

##String.raw()
   String.raw方法，往往用来充当模板字符串的处理函数，返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字符串。

    String.raw`Hi\n${2+3}!`; // "Hi\\n5!"

   String.raw方法也可以作为正常的函数使用。这时，它的第一个参数，应该是一个具有raw属性的对象，且raw属性的值应该是一个数组。

    String.raw({ raw: 'test' }, 0, 1, 2); // 't0e1s2t', 等同于 String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);

读阮一峰老师《ECMAScript6入门》一书的笔记    
<br>