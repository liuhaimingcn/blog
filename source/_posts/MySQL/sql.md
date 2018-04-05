title: sql 操作
date: 2016-5-26  
tags:
    - original
categories:
    - MySQL
---

删除重复的数据

```sql
delete a from tb_dict a
where (a.keyword,a.nature) in (
	select * from (select keyword,nature from tb_dict group by keyword, nature having count(*) > 1) b
) 
and a.id not in (
	select * from (select min(id) from tb_dict group by keyword, nature having count(*)>1) c
)
```

<!-- more -->      

查询重复的数据

```sql
select * from tb_dict a 
where (a.keyword,a.nature, a.freq) in (
	select keyword,nature, freq from tb_dict group by keyword, nature, freq having count(*) > 1
)
```



<br>