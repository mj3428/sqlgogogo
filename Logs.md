# MYSQL日志
## 二进制日志（binlog）
二进制日志（binlog）记录了所有的DDL（数据定义语言）语句和DML（数据操纵语言）语句，并以“事件”形式保存，对于灾难时的数据恢复起着极其重要
的作用。  
当用--log-bin[=file_name]选项启动时，mysqld开始将数据变更情况写入日志文件。如果没有给出file_name值，默认名为主机名后面跟"-bin"。
如果给出了文件名，但没有包含路径，则文件默认被写入参数DATADIR。
## 日志的读取
语法:`shell> mysqlbinlog log-file;`
## 日志的删除
* 方法1："RESET MASTER"命令  
* 方法2："PURGE MASTER LOGS TO 'mysql-bin.******'"命令，该命令将删除"******"编号之前的所有日志。  
* 方法
