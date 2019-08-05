# 优化数据库对象
## 优化表的数据类型
可以使用PROCEDURE ANALYSE()对当前应用的表进行分析，该函数可以对数据表中列的数据类型提出优化建议，用户可以根据应用的实际情况斟酌考虑
是否实施优化。使用方法为：
```
SELECT * FROM tbl_name PROCEDURE ANALYSE();
SELECT * FROM tbl_name PROCEDURE ANALYSE(16,256);
```
第二句告诉PROCEDURE ANALYSE()不要为那些包含的值多余16个或者256个字节的ENUM类型提出优化建议。如果没有这样的限制，输出信息可能很长；
ENUM定义通常很难阅读。  
