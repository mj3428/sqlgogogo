# 事务控制和锁定语句
## LOCK TABLE和UNLOCK TABLE
LOCK TABLES可以锁定用于当前线程的表。如果被其他线程锁定，则当前线程会等待，直到可以获取所有锁定为止。  
UNLOCK TABLES可以释放当前线程获得的任何锁定。当前线程执行另一个LOCK TABLES时，或当与服务器的连接被关闭时，所有由当前线程锁定的表被
隐含地解锁。  
*获得表film_text的READ锁定*
```
lock table filem_text read;
```
## 事务控制
*语法*
```
SSTART TRANSACTION|BEGIN [WORK]
COMMIT[WORK][AND [NO] CHAIN][[NO] RELEASE]
ROLLBACK [WORK][AND [NO] CHAIN][NO [RELEASE]]
SET AUTOCOMMIT={0|1}
```
* start transaction或begin语句可以开始一项新的事务  
* commit和rollback用来提交或者回滚事务
* chain和RELEASE子句分别用来定义在事务的提交或者回滚之后的操作，chain会立即启动一个新事务，并且和刚才的事务具有相同的隔离级别， 
  Release则会断开和客户端的连接  
* set autocommit可以修改当前连接的提交方式，如果设置了SET AUTOCOMMIT=0，则设置之后的所有事务都需要通过明确的命令进行提交或者回滚  

在同一个事务中，最好不使用不同存储引擎的表，否则ROLLBACK时需要对非事务类型的表进行特别的处理，因为COMMIT,ROLLBACK只能对事务类型的表
进行提交和回滚。  

在事务中可以通过定义SAVEPOINT，指定回滚事务的一个部分，但是不能指定提交事务的一个部分。对于复杂的应用，可以定义多个不同的SAVEPOINT,
满足不同的条件是，回滚不同的SAVEPOINT。当然，如果定义了相同名字的SAVEPOINT，则后面的定义的SAVEPOINT会覆盖之前的定义。对于不再需要
使用的SAVEPOINT,可以通过RELEASE SAVEPOINT命令删除SAVEPOINT，删除后的SAVEPOINT不能再执行ROLLBACK TO SAVEPOINT命令。
## 分布式事务
*语法*
`
XA {STARR|BEGIN} xid [JOIN|RESUME]
`
每个事务不许有一个唯一的xid值，该值当前不能被其他的XA事务使用。xid是一个XA事务标识符，用来唯一标识一个分布式事务。  
xid:gtrid[,buqual[,formatID]]  
* gtrid 是一个分布式事务标识符，相同的分布式事务应该使用相同的gtrid，这样可以明确知道XA事务属于哪个分布式事务。  
* bqual 是一个分支限定符，默认是空串。对于一个分布式事务中的每个分支事务，bqual值必须是唯一的。  
* formatID 是一个数字，用于标识由gtrid和bqual值使用的格式，默认值是1。  
*其他语句*  
```
XA END xid [SUSPEND [FOR MIGRATE]]
XA PREPARE xid
# 使事务进入PREPARE状态，也就是两阶段提交的第一个提交阶段

XA COMMIT xid [ONE PHASE]
XA ROLLBACK xid
# 上面两个命令用来提交或者回滚具体的分支事务。

XA RECOVER
XA RECOVER返回当前数据库中出PREPARE桩体的分支事务的详细信息。
```
