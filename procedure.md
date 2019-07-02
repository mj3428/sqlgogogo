## 变量的使用
1. 变量的定义    
   通过DECLARE可以定义一个局部变量，该变量的作用范围只能在BEGIN..END块中，可以用在嵌套的块中。变量的定义必须写在复合语句的开头
   ，并且在任何其他语句的前面。语法如下:  
   ```
   DECLARE var_name[...]type[DEFAULT value]
2. 变量的赋值  
   变量可以直接赋值，或者通过查询赋值。直接赋值使用SET，可以赋值常量或者赋表达式。  
   *例：通过查询结果赋值给变量v_payments*  
   ```
   CREATE FUNCTION get_customer_balance(p_customer_id INT,p_effective_date DATETIME)
   RETURNS DECIMAL(5,2)
   DETERMINISTIC
   READS SQL DATA
   BEGIN
    ...
    DECLARE v_payments DECIMAL(5,2); #SUM OF PAYMENTS MADE PREVIOUSLY
    ...
    SELECT IFNULL(SUM(payment.amount), 0) INTTO v_payments
    FROM payment
    WHERE payment.payment_date <= p_effective_date
    AND payment.customer_id = p_customer_id;
    ...
    RETURN v_rentfees + v_overfees - v_payments;
   END $$
   ```
## 定义条件和处理
```
delimiter $$
CREATE PROCEDURE actor_insert()
BEGIN
   DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @x2 = 1;
   SET @x = 1;
 INSERT INTO actor(actor_id,first_name,last_name) VALUES (201,'Test','201');
   SET @x = 2;
 INSERT INTO actor(actor_id,first_name,last_name) VALUES (1,'Test','1');
   SET @x = 3;
END
$$
```
写成以下几种方式：
```
--捕获mysql-error-code:
DECLARE CONTINUE HANDLER FOR 1062 SET @x2 = 1;
--事先定义condition_name:
DECLARE DuplicateKey CONDITION FOR SQLSTATE '23000';
DECLARE CONTINUE HANDLER FOR DuplicateKey SET @X2 = 1;
--捕获SQLEXCEPTION
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET @x2 =1;
```
## 光标的使用
* 声明光标  
  `DECLARE cursor_name CURSOR FOR select_statement`
* OPEN光标  
  `OPEN cursor_name`
* FETCH光标  
  `FETCH cursor_name INTO var_name[, var_name]..`
* CLOSE光标  
  `CLOSE cursor`
