# 插入优化

1. 如果能批量插入那就尽量批量插入；存储过程中的批量插入可以用事务来提高速度；
2. 批量插入前先disable索引，插入完成后再enable索引；
   * ALTER TABLE table_name DISABLE KEYS;
   * 批量插入;
   * ALTER TABLE table_name ENABLE KEYS;
3. 提高innodb_buffer_pool_size；
4. 设置innodb_flush_log_at_trx_commit为2，每秒刷入磁盘。

## 例子
优化一个存储过程，将其运行时间从1.5小时降到10秒，方法是开启事务。

```SQL
CREATE DEFINER=`root`@`%` PROCEDURE `optimize_train`.`generate_orders`()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 1000; -- 用户数量
    DECLARE total_orders_per_user INT DEFAULT 1000; -- 每个用户的订单数量
    DECLARE rnd_user_id INT;
    DECLARE rnd_order_date DATE;
    DECLARE rnd_total_amount DECIMAL(10, 2);
    DECLARE j INT DEFAULT 0;

    WHILE i < total_users DO
        -- 获取用户ID
        SELECT user_id INTO rnd_user_id FROM users LIMIT i, 1;

        START TRANSACTION; -- <-开启事务
        WHILE j < total_orders_per_user DO
            -- 生成订单日期和总金额
            SET rnd_order_date = DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 1096) DAY); -- 2020-01-01和2022-12-31之间的随机日期
            SET rnd_total_amount = ROUND(RAND() * 1000, 2); -- 0到1000之间的随机总金额
            -- 将数据插入订单表
            INSERT INTO orders (user_id, order_date, total_amount) VALUES (rnd_user_id, rnd_order_date, rnd_total_amount);

            SET j = j + 1;
        END WHILE;
        COMMIT;
       
        SET j = 0;

        SET i = i + 1;
    END WHILE;
END
```
