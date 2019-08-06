# 锁问题
## 概述
MyISAM和MEMORY存储引擎采用的是表级锁(table-level locking),BDB存储引擎采用的是页面锁(page-level locking),但也支持表级锁，
InnoDB存储引擎及支持行级锁(row-level locking)，也支持表级锁，但默认情况下采用行级锁。  
- 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度低。  
- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度大最小，发生锁冲突的概率最低，并发度也最高。  
- 页面锁：开销和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般。  
## 表加锁
```
select sum(total) from orders;
select sum(subtotal) from order_detail;
```
一般，如果不先给两个表加锁，就可能产生错误的结果，因为第一条语句执行过程中，order_detail表可能已经发生了改变。正确方法如下：
```
lock tables orders read local,order_detail read local;
select sum(total) from orders;
select sum(subtotal) from order_detail;
unlock tables;
```
在执行lock tables后，智能访问显式加锁的这些表，不能访问未加锁的表；同时，如果加的是读锁，那么智能执行查询操作，而不能执行更新操作。  
## 什么时候使用表锁
1. 使用LOCK tables 虽然可以给INNODB加表级锁，但必须说明的是，表锁不是由INNODB存储引擎层管理的，而是由其上一层MYSQL Server负责的
仅当autocommit=0,innodb_table_locks=1时，INNODB层才知道MYSQL加的表锁，MYSQL SERVER也才能感知INNODB加的行锁，这种情况下，INNODB才能
自动识别涉及表级锁的死锁；否则，INNODB将无法自动检测并处理这种死锁。  
2. 在用LOCK TABLES对INNODB表加锁时要注意，要将AUTOCOMMIT设为0，否则MYSQL不会给表加锁；事务结束前，不要用UNLOCK TABLES释放表锁，因为
UNLOCK TABLES会隐含地提交事务；COMMIT或ROLLBACK并不能释放用LOCK TABLES加的表级锁，必须用UNLOCK TABLES释放表锁。例如：
```
set autocommit=0;
lock tables t1 write, t2 read,...;
[do something with tables t1 and t2 here];
commit;
unlock tables;
```
