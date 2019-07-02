# 触发器
## 创建触发器
```
CREATE TRIGGER trigger_name triger_time trigger_event ON tbl_name FOR EACH ROW trriger_stmt
```
*注：* 触发器只能创建在永久表上，不能对临时表创建触发器  
trigger_time是触发器的触发时间，可以AFTER/BEFORE,BEFORE指在检查约束前触发，AFTER在检查约束后触发。
*为film表创建了AFTER INSERT的触发器*
```
DELIMITER $$
CREATE TRIGGER ins_film
AFTER INSERT ON film FOR EACH ROW BEGIN
  INSERT INTO film_text (film_id, title, description)
    VALUE (new.film_id, new.title, new.description);
END
$$
delimiter;
```
