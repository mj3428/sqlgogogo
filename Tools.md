# MYSQL中的常用工具
## mysql（客户端连接工具）
一般在终端写的指令。  
两种表达方式，一种是"-"+选项单词的缩写字符+选项值；另一种是"--"+选项的完完整单词+"="+选项的实际值。  
* **选项连接**
```
-u, --user=name 指定用户名
-p, --password[=name] 指定密码
-h, --host=name 指定服务器IP或者域名
-P, --port=# 指定连接端口
```
通过select current_user()查看当前的连接用户。  
也可以进行远程连接，比如`mysql -h 192.168.7.55 -P 3306 -uroot -p`当然一般也不会去获取root权限，端口也可以更改。  
* **执行选项**  
脚本可以通过类似`mysql -u -root -p -e "SELECT Name FROM Country WHERE Name Like 'AU%';SELECT COUNT(*) FROM City"`来执行语句，不同语句
之间通过分号（;）隔开。  
* **错误处理选项**  
```
-f, --force 强制执行SQL
-v, --verbose 显示更多信息
--show-warnings 显示警告信息
```
* **myisampack表压缩工具**  
用法如下:`shell> myisampack [options] filename`
* **mysqladmin管理工具**  
用法如下:`shell> mysqladmin [options] command [command-options][command[command-options]]`  
比如`shell# mysqladmin -uroot -p shutdown`  
* **mysqlbinlog日志管理工具**  
用法如下：`shell> mysqlbinlog[options] log-files1 log-files2...`选下如下：   
  - -d,--database=name:指定数据库名称，只列出指定的数据库相关操作；  
  - -o,--offset=#:忽略掉日志中的前n行命令；  
  - -r,--result-file=name:将输出的文本格式日志输出到指定文件；  
  - -s,--short-form：显示简单个事，省略掉一些信息；  
  - --set-charset=char-name:在输出为文本格式时，在文件第一行加上set names char-name,这个选项在某些情况下装在数据时非常有用；  
  - --start-datetime=name-stop-datettime=name:指定日期间隔内的所有日志；  
  - --start-position=# --stop-position=#: 指定位置间隔内的所有日志。  
* **mysqlcheck表维护工具**  
用法如下:
```
shell> msyqlcheck [options] db_name [tables]
shell> mysqlcheck [options] --database DB1 [DB2 DB3..]
shell> mysqlcheck [options] --all--database
```
option选项如下：  
  - -c, --check(检查表)；  
  - -r, --repair(修复表)；  
  - -a, --analyze(分析表)；  
  - -o, --optimize(优化表)；  
* **mysqldump数据导出工具**
用法如下：
```
shell> mysqldump [options] db_name #备份单个数据库或者库中部分数据表
shell> mysqldump [options] --database DB1[DB2 DB3] #备份指定的一个或多个数据库
shell> mysqldump [options] --all--database #备份所有数据库
```
  * **输出内容选项**  
  > --add-drop-database 每个数据库创建语句前加上DROP DATABASE语句  
  > --add-drop-table 在每个表创建语句前DROP TABLE语句  
  > -n, --no-create-db 不包含数据的创建语句  
  > -t, --no-create-info 不包含数据的创建语句  
  > -d, --no-data 不包含数据  
* **mysqlshow数据库对象查看工具**  
使用方法:`shell> msyqlshow[option][db_name[tal_name[col_name]]]`  
* **replace文本替换工具**
```
shell> replace from to [from to].. --file[file]..
shell> replace from to [from to].. <file
```

覆盖方式(--);非覆盖方式(<);
