# 查询优化案例2

## 创建表
```SQL
CREATE TABLE `orders` (
  `order_id` int NOT NULL AUTO_INCREMENT,
  `user_id` int DEFAULT NULL,
  `product_id` int DEFAULT NULL,
  `order_date` date NOT NULL,
  `total_amount` decimal(10,2) NOT NULL,
  PRIMARY KEY (`order_id`)
  -- KEY `idx_user_id` (`user_id`) USING BTREE
  -- KEY `idx_user_amount` (`user_id`,`total_amount`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;


CREATE TABLE `users` (
  `user_id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_general_ci NOT NULL,
  `email` varchar(100) COLLATE utf8mb4_general_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`)
  -- KEY `idx_user_id` (`user_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

CREATE TABLE `products` (
    `product_id` INT NOT NULL AUTO_INCREMENT,       -- 产品ID，主键，自增长
    `name` VARCHAR(255) NOT NULL,               -- 产品名称
    `description` TEXT,                         -- 产品描述
    `price` DECIMAL(10, 2) NOT NULL,            -- 产品价格
    `stock_quantity` INT NOT NULL,              -- 库存数量
	 PRIMARY KEY(`product_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

## 生成数据
```SQL
CREATE PROCEDURE optimize_train_2.create_orders()
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_users INT DEFAULT 10000; -- 用户数量
    DECLARE total_orders_per_user INT DEFAULT 1000; -- 每个用户的订单数量
    DECLARE rnd_user_id INT;
    DECLARE rnd_product_id INT;
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
            SET rnd_product_id = ROUND(RAND() * 100000, 0); 
            -- 将数据插入订单表
            INSERT INTO orders (user_id, product_id, order_date, total_amount) VALUES (rnd_user_id, rnd_product_id, rnd_order_date, rnd_total_amount);

            SET j = j + 1;
        END WHILE;
        COMMIT;
       
        SET j = 0;

        SET i = i + 1;
    END WHILE;
END

CREATE PROCEDURE optimize_train_2.create_products()
BEGIN
	DECLARE i INT DEFAULT 0;
    DECLARE total_products INT DEFAULT 100000; -- 用户数量

    DECLARE rnd_user_id INT;
    DECLARE rnd_product_id INT;
    DECLARE rnd_product_price DECIMAL(10, 2);
    DECLARE rnd_product_name VARCHAR(50);
    DECLARE rnd_stock_quantity INT;

    WHILE i < total_products DO
        -- 生成随机用户名和邮箱
        SET rnd_product_name = CONCAT('Prod', FLOOR(1 + RAND() * 10000000)); -- 假设用户名唯一
		SET rnd_product_price = ROUND(RAND() * 1000, 2);
	    SET rnd_stock_quantity = ROUND(RAND() * 1000, 0);
        -- 将数据插入用户表
        INSERT INTO products (name, price, stock_quantity) VALUES (rnd_product_name, rnd_product_price, rnd_stock_quantity);

        SET i = i + 1;
    END WHILE;
END

CREATE PROCEDURE optimize_train_2.create_users()
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
```
