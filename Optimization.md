# 优化
[案例库sakila](http://downloads.mysql.com/docs/sakila-db.zip)
## 技巧
- 比较运算符能用 “=”就不用“<>”  

- 明知只有一条查询结果，那 请使用 “LIMIT 1”;  
  “LIMIT 1”可以避免全表扫描，找到对应结果就不会再继续扫描了。  
  
- 为列选择合适的数据类型  
  能用TINYINT就不用SMALLINT，能用SMALLINT就不用INT  
  
- 将大的DELETE，UPDATE or INSERT 查询变成多个小查询  

- 使用UNION ALL 代替 UNION，如果结果集允许重复的话;  
  UNION ALL 不去重，效率高于 UNION;  
  
- 为获得相同结果集的多次执行，请保持SQL语句前后一致  

- 尽量避免使用 “SELECT *”  

- WHERE 子句里面的列尽量被索引;  
  因地制宜，根据实际情况进行调整，因为有时索引太多也会降低性能。  
  
- JOIN 子句里面的列尽量被索引  

- ORDER BY 的列尽量被索引  

- 使用 LIMIT 实现分页逻辑  

- 使用 EXPLAIN 关键字去查看执行计划  
### 通过show status命令了解各种SQL的执行频率
`show status like 'com_%'`  
com_xxx表示每个xx语句执行的次数，我们通常比较关心的是：  
com_select:执行SELECT操作的次数，一次查询只累加1；  
com_insert:执行INSERT操作的次数，一次查询只累加1,批量插入，只累加一次；  
com_update:执行UPDATE操作的次数
com_delete:执行DELETE操作的次数
...
### 定位执行效率低的sql语句
- 用--log-slow-queries[=file_name]选项启动是，mysqld写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件  
- 用show processlist命令查看当前mysql在进行的线程，包括线程的状态、是否锁表等。  
### 通过EXPLAIN分析低效SQL的执行计划
select查询前加上explain关键词；但是select也有其select_type，也有常见的类型

| ALL | index | range | ref | eq_ref | const,system | NULL |
| :--- | :---| :--- | :--- | :--- | :--- | :--- |  
**从左到右，性能逐渐变好。**
比如：  
`explain select * from film where rating > 9\G`（\G为结束符）  
这个语句type就是为all;  
`explain select title from film`  
这个是title为index,所以为第二类，用索引找，相较于全表扫描，更快;  
接下来看range:  
`explain select * from payment where custmoer_id >= 300 and customer_id <= 350`用索引来选取范围；  
接下来看type=ref:  
`explain select * from payment where customer_id=350\G`这里type=ref,key=idx_fk_customer_id,ref=const;  
**索引idx_fk_customer_id是非唯一索引，查询条件为等值，所以扫描类型为ref**
接下来看type=eq_ref:  
`explain select * from film a, film_text b where a.film_id = b.film_id\G`多表连接中使用primary key或者unique index
作为关联条件。这里type=eq_ref,key=primary。  

