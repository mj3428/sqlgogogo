# MYSQL 分区
## 分区类型
* RANGE分区：给予一个给定连续区间范围，把数据分配到不同的分区。  
* LIST分区：类似RANGE分区，区别在LIST分区是基于枚举出的值列表分区，RANGE是基于给定的连续区间范围分区。  
* HASH分区：基于给定的分区个数，把数据分配到不同的分区。  
* KEY分区：类似于HASH分区。  
无论是哪种分区，要么分区表上没有主键/唯一键，要么分区表的主键/唯一键都必须包含分区键，也就是说不能使用主键/唯一键字段之外的其他字段分区。  
## RANGE分区
按照ARANGE分区的表示利用取值范围将数据分成分区，区间要连续并且不能互相重叠，使用VALUES LESS THAN操作进行分区定义。  
*例，雇员表emp中按商店ID store_id进行RANGE分区*
```
CREATE TABLE emp(
  id INT NOT NULL,
  ename VARCHAR(30),
  hired DATE NOT NULL DEFAULT '1970-01-01',
  separated DATE NOT NULL DEFAULT '9999-12-31',
  job VARCHAR(30) NOT NULL,
  store_id INT NOT NULL,
)
PARTITION BY RANGE (store_id)(
  PARTITION p0 VALUES LESS THAN (10),
  PARTITION p1 VALUES LESS THAN (20),
  PARTITION p2 VALUES LESS THAN (30),
);
```
