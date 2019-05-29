# 优化
## 技巧
- 比较运算符能用 “=”就不用“<>”  

- 明知只有一条查询结果，那 请使用 “LIMIT 1”;  
  “LIMIT 1”可以避免全表扫描，找到对应结果就不会再继续扫描了。  
  
- 为列选择合适的数据类型  
  能用TINYINT就不用SMALLINT，能用SMALLINT就不用INT  
  
- 将大的DELETE，UPDATE or INSERT 查询变成多个小查询  

- 使用UNION ALL 代替 UNION，如果结果集允许重复的话;  
  UNION ALL 不去重，效率高于 UNION;  
  
- 为获得相同结果集的多次执行，请保持SQL语句前后一致  

- 尽量避免使用 “SELECT *”  

- WHERE 子句里面的列尽量被索引;  
  因地制宜，根据实际情况进行调整，因为有时索引太多也会降低性能。  
  
- JOIN 子句里面的列尽量被索引  

- ORDER BY 的列尽量被索引  

- 使用 LIMIT 实现分页逻辑  

- 使用 EXPLAIN 关键字去查看执行计划  
