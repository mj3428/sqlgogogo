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
接下来看type=const/system，但表中最多有一个匹配行，查询起来迅速，比如根据主键primary key或者unique index进行查询:  
```
先设置主键
alter table customer drop index idx_email;
alter table customer add unique index uk_email(email);
再查询
explain select * from (select * from customer where email = 'AARON.SELBY@sakilacustomer.org')a\G
```
这里type=const,key=uk_email
### 通过show profile分析sql
可以通过set语句在Session级别开启profiling:`set profiling=1`  
BTREE是最常见的 索引，构造类似二叉树，B不代表二叉树(binary)，而是代表平衡(balanced)  
### 索引扫描
- **复合索引**的情况下，假如查询条件不包含索引列最左边部分，即不满足最左原则，是不会使用复合索引的  
- 如果Mysql估计使用索引比全表扫描更慢，则不使用索引，有的时候如，查询以“S”开头的标题电影，需要返回的记录比例较大，mysql就预估索引扫描还不如
  全表扫描更快，就会全表扫描。**有的时候，我们可以用trace scan，再通过rows和cost计算扫描的成本，就能比效率了**  
- InnoDB中表上二级索引，理想的访问方式应该是首先扫描二级索引获得满足条件的主键列表，之后根据主键回表去检索记录，这样访问不开了全表扫描产生的
  的大量IO请求。  
- **查看索引使用情况:** handler_read_key的值将很高，这个值代表了一个行被索引读的次数，很低的值表明增加索引得到的性能改善不高，因为
  索引并不经常使用；handler_read_rnd_next的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的
  请求数。语句为`show status like 'Handler_read%'`
### 两个简单实用的优化方法
#### 1）定期分析表和检查表
* 分析表语句：`ANALYZE [LOCAL|NO_WARITE_TO_BINLOG] TABLE tbl_name[,tbl_name]...`比如analyze table payment  
* 检查表语句: `CHECK TABLE tbl_name[,tbl_name]..[option]..option = {QUICK|FAST|MEDIUM|EXTENDED|CHANGED}`  
#### 2)定期优化表
* 优化表语句： `OPTIMIZE [LOCAL|NO_WRITE_TO_BINLOG] TABLE tbl_name [,tbl_name]...`  
### 常用的SQL优化
1. 大批量插入数据
> 对于MyISAM存储引擎的表可以通过以下语句导入大量数据
```
ALTER TABLE tbl_name DISABLE KEYS;
loading the data
ALTER TABLE tbl_name ENABLE KEYS;
```
> DISABLE KEYS和ENABLE KEYS用来打开或者关闭MyISAM表非唯一索引的更新。在导入大量的数据到一个费控的MyISAM表时，通过设置这两个命令，
可以提高导入效率。也就是说**导入数据前，关闭索引！！！**  
*举例:*
```
i
load data infile 'home/mysql/film_test.txt' into table film_test2;
耗时115.12
ii
alter table film_test2 disable keys;
load data infile 'home/mysql/film_test.txt' into table film_test2;
alter table film_test2 enable keys;
耗时18.59秒
```
> 另外，InnoDB类型是表时按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效的提高导入数据的效率。  
> 在导入数据前执行SET UNIQUE_CHECKS=0,关闭唯一性校验，在导入结束后执行SET QUNIQUE_CHECKS=1恢复唯一性校验。  
*举例:*
```
i UNIQUE_CHECKS=1时
load data infile '/home/mysql/film_test3.txt' into table film_test4;
耗时22.92秒
ii UNIQUE_CHECKS=0时
load data infile '/home/mysqlfilm_test3.txt' into table film_test4;
耗时19.92秒
```
> 建议在导入前执行SET AUTOCOMMIT=0，关闭自动提交，导入结束后再执行SET AUTOCOMMIT=1,打开自动提交，也可以提高导入效率  
2. 优化INSERT语句
> 尽量一次插入多个值，如`insert into test values(1,2),(1,3),(1,4)...`  
> 如果从不同客户插入多行.可以通过使用INSERT DELAYED语句得到更高的速度。DELAYED含义是让INSERT语句马上执行。  
3. 优化ORDER BY语句  
> explain中有一个extra的类型，显示为using filesort；Filesort是通过相应的排序算法，将取得的数据在sort_buffer_size
系统变量设置的内存排序区中进行排序，如果内存装载不下，它就会将磁盘上的数据进行分块，再对各个数据块进行排序，然后将各个块合并
成有序的结果集。ssort_buffer_size设置的排序区是每个线程独占的，所以同一个时刻，Mysql中存在多个sort buffer排序区。
**总的来说Filesort非常耗时**  
> 下列SQL可以使用索引:
```
SELECT * FROM tabname ORDER BY key_part1, key_part2,...;
SELECT * FROM tabname WHERE key_part1=1 ORDER BY key_part1 DESC, key_part2 DESC;
SELECT * FROM tabname ORDER BY key_part1 DESC, key_part2 DESC;
```
> 但是在以下几种情况下则不适用索引：
```
SELECT * FROM tabname ORDER BY key_part1 DESC, key_part2 ASC;
-- order by的字段混合ASC和DESC
SELECT * FROM tabname WHERE key2=constant ORDER BY key1;
-- 用户查询行的关键字与ORDER BY中所使用的的不相同
SELECT * FROM tabname ORDER BY key1, key2;
-- 对不同的该关键字使用ORDER BY;
```
> **Filesort的优化**通过创建合适的索引能够减少Filesort出现，但是在某些情况下，条件限制不能让Filesort小时，那就需要想办法
加快Filesort的操作。适当加大max_length_for_sort_data的值，能够让MySQL选择更优化的Filesort排序算法  
4. 优化GROUP BY语句
> GROUP BY之后添加ORDER BY NULL，禁止聚合后的Filesort  
5. 优化嵌套查询  
> 使用JOIN连接来完成查询，速度会快很多，尤其建有索引的情况。
6. 优化分页查询
> 第一种避免全表扫描,错误示范:`select film_id,description from film order by title limit 50,5`;再换个思路
`select a.film_id, a.discription from film a inner join (select film_id from film order by title limit 50,5)b on
a.film_id=b.film_id`
> 第二种，记住页码，和分页最后一个数据，然后倒着来取数据，比如取41页10个数据，就记住42页第一个数据x，小于x,再limit。简单讲
  就是把LIMIT m,n转换成LIMIT n的查询。**但是适合在排序字段不会出现重复值的特定环境，但是有出现大量重复值，而进行优化，有
  丢失分页记录的风险**
