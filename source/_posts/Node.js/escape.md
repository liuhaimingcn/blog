title: Node.js中用escape解决sql注入
date: 2016-7-5  
tags:
    - original
categories:
    - Node.js
---

### 直接拼写sql进行数据库的操作时，很容易被人在动态参数中加入特殊字符产生sql注入，威胁数据库的安全。

``` js
'use strict';

const mysql = require('mysql');

let param = 'ns';
let pool = mysql.createPool({
  user: 'root',
  password: 'root',
  database: 'nlp_dict'
});

pool.getConnection(function (err, conn) {
  let sql = 'select * from tb_nature where nature = "' + param + '" and del_status=1';
  conn.query(sql, function (err, result) {
    console.log(result);
  })
});
```
<!-- more -->

这时正常情况下能查询到一条数据，如果将param修改成  
let param = 'ns"-- ';  
sql语句就会变成  
select * from tb_nature where nature = "ns"-- " and del_status=1  
后面的del_status就会被参数中的 -- 注释掉，失去作用，能查询到多条数据。  

如果对param使用escape包装下，就能将参数中的特殊字符进行转义，防止sql的注入。
let sql = 'select * from tb_nature where nature = ' +  mysql.escape(param) + ' and del_status=1';

<br>