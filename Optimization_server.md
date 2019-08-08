# 优化MYSQL SERVER
## MYSQL体系结构
默认情况下，MYSQL有7组后台线程，分别是1个主线程，4组IO线程，1个锁线程，1个错误监控线程，1个purge线程。  
- master thread:主要负责将脏缓存页刷新到数据文件，执行purge操作，触发检查点，合并插入缓冲区。  
- insert buffer thread:负责数据库插入缓冲区的合并操作。  
- read thread:负责数据库读取操作，可配置多个读线程。  
- write thread:负责数据库写操作，可配置多个写线程。  
- log thread:用于将重做日志刷新到logfile中。  
- purge thread:执行purge操作。  
- lock thread:负责锁控制和死锁检测等。  
- 错误监控线程：主要负责错误监控和错误处理。  
通过`show engine innodb status`查看线程状态
## 内存管理及优化
- MYISAM的数据文件读取依赖于操作系统自身的IO缓存，因此，如果有MYISAM表，就要预留更多的内存给操作系统做IO缓存。  
- 排序区、连接区等缓存时分配每个数据库会话(session)专用的，其默认值要根据最大连接数合理分配，如果设置太大，不但浪费内存资源，
  而且在并发连接较高时会导致物理内存耗尽。  
## InnoDB内存优化
* **缓存机制**  
InnoDB用一块内存区做IO缓存池，该缓存池不仅用来缓存InnoDB的索引块，而且用来缓存InnoDB的数据块。  
LRU list是InnoDB正在使用的缓存块，原理类似MYISAM的中点插入策略，详细为：LRU list分为young sublist和old sublist，数据从
磁盘读入时，会将该缓存块插入到LRU list的“中点”，即old sublist的头部；经过一定时间的访问（由innodb_old_blocks_time系统参数
决定），该数据块将会由old sublist转移到young list的头部，也就是整个LRU list的头部；随着时间的推移，young list和old sublist
中较少被访问的缓存块将从各自链表的头部逐渐向尾部移动；需要淘汰数据块时，优先从链表尾部淘汰。同样也是为了防止偶尔被访问的索引
块将访问频繁的热块淘汰。  
脏页的刷新存在于flush list和LRU list这两个链表，LRU上也存在可以刷新的脏页，这里是直接可以刷新的，默认BP中不存在可用的数据页
的时候会扫描LRU list尾部的innodb_lru_scan_depth个数据页（默认为1024个数据页）。从LRU list淘汰的数据页会立刻放入到free list中去。
* **innodb_buffer_pool_size**  
innodb_buffer_pool_size决定InnoDB存储引擎表数据和索引数据的最大缓存区大小。和InnoDB buffer pool同时为数据块和索引块提供数据缓存，
这与Oracle的缓存机制很相似。在保证操作系统及其他程序有足够内存可用的情况下，innodb_buffer_pool_size的值越大，缓存命中率越高，
访问InnoDB表需要的磁盘IO就越少，性能也就越高。查看buffer pool的使用情况的命令`mysqladmin -S /tmp/mysql.sock ext|grep -i
 innodb_buffer_pool`  
 **InnoDB缓存池的命中率计算公式：（1-innodb_buffer_pool_reads/innodb_buffer_pool_read_request）×100** 如果命中率太低，则应考虑
 扩充内存、增加xx_pool_size的值。
 * **调整old sublist大小**
