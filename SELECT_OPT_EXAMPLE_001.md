# 查询优化案例1

## 创建表
```SQL
CREATE TABLE `orders` (
  `order_id` int NOT NULL AUTO_INCREMENT,
  `user_id` int DEFAULT NULL,
  `order_date` date NOT NULL,
  `total_amount` decimal(10,2) NOT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

CREATE TABLE `users` (
  `user_id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `email` varchar(100) COLLATE utf8mb4_general_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

## 生成数据
```SQL
-- 生成用户数据，一共10000个用户
CREATE DEFINER=`root`@`%` PROCEDURE `optimize_train`.`create_users`()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 10000; -- 调整用户数量
    DECLARE rnd_username VARCHAR(50);
    DECLARE rnd_email VARCHAR(100);

    WHILE i < total_users DO
        -- 生成随机用户名和邮箱
        SET rnd_username = CONCAT('User', FLOOR(1 + RAND() * 10000000)); -- 假设用户名唯一
        SET rnd_email = CONCAT(rnd_username, '@example.com'); -- 假设邮箱唯一
        -- 将数据插入用户表
        INSERT INTO users (username, email) VALUES (rnd_username, rnd_email);

        SET i = i + 1;
    END WHILE;
END

-- 创建订单数据，一共一千万
CREATE DEFINER=`root`@`%` PROCEDURE `optimize_train`.`generate_orders`()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 10000; -- 用户数量
    DECLARE total_orders_per_user INT DEFAULT 1000; -- 每个用户的订单数量
    DECLARE rnd_user_id INT;
    DECLARE rnd_order_date DATE;
    DECLARE rnd_total_amount DECIMAL(10, 2);
    DECLARE j INT DEFAULT 0;

    WHILE i < total_users DO
        -- 获取用户ID
        SELECT user_id INTO rnd_user_id FROM users LIMIT i, 1;

        START TRANSACTION;
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
