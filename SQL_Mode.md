# SQL Mode
## mysql sqlmode功能
* 通过设置sqlmode可以完成不同严格程度的数据校验，有效地保障数据准确性  
* 通过设置sqlmode为ANSI模式，来保证大多数SQLL符合标准的SQL语法，这样应用在不同数据库之间进行迁移时，则不需要对业务SQL进行较大的修改。  
* 在不同数据库之间进行数据库迁移之前，通过设置SQLmode可以使MYSQL上的数据方便地迁移到目标数据库中。  

*查看默认的sqlmode的命令：*
`
select @@sql_mode
`  
默认有：REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,GNORE_SPACE和ANSI。
