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
## List分区
离散的值分区
和Range分区的区别：List分区是从属于一个枚举列表的值的集合，Range分区是从属于一个连续区间值的集合，表达方式：PARTITION BY LIST(expr)
然后使用VALUES IN(value_list)的方式来定义分区，其中value_list是一个逗号分隔的整数列表。  
```
CREATE TABLE expenses(
expense_date DATE NOT NULL,
category INT,
amount DECIMAL (10,3)
)PARTITION  BY LIST(category)(
  PARTITION p0 VALUES IN (3, 5),
  PARTITION p1 VALUES IN (1, 10),
  PARTITION p2 VALUES IN (4, 9),
  PARTITION p3 VALUES IN (2),
  PARTITION p4 VALUES IN (6),
);
```
## Columns分区
Columns分区增加数据类型：  
- 所有整数类型：tinyint,smallint,mediumint,int 和 bigint;但不支持decimal和float  
- 日期时间类型:date和datetime  
- 字符类型：char,varchar,binary,varbinary;不支持text和bblob类型作为分区键  
其实，RANGE Columns分区键的比较其实就是多列排序，先根据a字段排序再根据b字段排序，根据排序结果来分区存放数据。和Range但字段分区排序的
规则实际上是一致的。  
## Hash分区
对分区键应用一个散列函数，以此确定数据应当放在N个分区中的哪个分区中。  
MySQL支持两种HASH分区，**常规HASH分区和线性HASH分区** 常规采用的是取模算法。线性HASH使用的是一个线性的2的幂的运算法则。  
表达方式：PARTITION  BY HASH(expr)PARTITIONS num子句对分区类型、分区键和分区个数进行定义，其中expr是某列值或表达式
num是一个非负整数，表示分割成分区的数量，默认num为1。  
```
CREATE TABLE emp(
id INT NOT NULL,
ename VARCHAR(30),
hired DATE NOT NULL DEFAULT '1970-01-01',
separated DATE NOTNULL DEFAULT '9999-12-31',
job VARCHAR(30) NOT NULL,
store_id INT NOT NULL
)
PARTITION BY HASH (store_id) PARTITIONS 4;
```
比如store_id = 234,MOD(234,4)=2,就会被分区到第二分区；  
线性HASH比常规在BY之前多一个LINEAR
线性分区方法:比如你分区分四个，V=Power(2, Ceiling(Log(2, num))), num=4;  
V=Power(2, Ceiling(2))=4  
线性HASH分区的有点是，在分区维护（包含增加、删除、合并、拆分分区）时，Mysql能够处理得更加迅速；缺点是，对比常规HASH
分区（取模）的时候，线性HASH各个分区之间数据的分布不太均衡。  
