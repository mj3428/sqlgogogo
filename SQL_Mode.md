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
*比如要设置的时候，可以*`set session sql_mode='STRICT_TRANS_TABLES`进入严格模式，字段类型就会不允许超出  
## sqlmode常见功能
1. 校验日期数据合法性
2. INSERT或UPDATE过程中，如果SQL MODE处于TRADITIONAL模式，那么运行MOD(X,0)就会产生错误，因为TRADITIONAL也属于严格模式；
   ，在非严格模式下MOD(X, 0)返回的结果是NULL。
3. 启用NO_BACKSLASH_ESCAPES模式，使反斜线成为普通字符。在导入数据时，如果数据中含有反斜线字符，那么启用NO_BACKSLASH_ESCAPES模式保证数据
   的正确性。
4. 启用PIPES_AS_CONCAT模式。将"||"视为字符串连接操作符。
