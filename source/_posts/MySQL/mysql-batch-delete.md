title: mysql 批量删除错误分析
date: 2016-5-26  
tags:
    - original
categories:
    - MySQL
---


```sql
delete t1 from tb_dict t1 
where t1.id in (
	select t2.id from tb_dict t2 where t2.nature = 'v-taxi'
);
```

使用这条 sql 语句进行批量删除时报 "You can't specify target table 't1' for update in FROM clause" 错误，查询后得知原来 msyql 不允许在子查询的同时删除原表中的数据。下面对应的解决办法。


```sql
delete t1 from tb_dict t1 
where t1.id in (
	select t3.id from (
		select * from tb_dict t2 where t2.nature = 'v-taxi'
	) as t3
);
```

将子查询得到的数据封装成临时表，这时就能解决问题了。


<br>