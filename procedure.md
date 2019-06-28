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
