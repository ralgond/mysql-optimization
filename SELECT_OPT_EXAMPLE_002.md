# 查询优化案例2

## 创建数据库

[创建脚本](CREATE_DB.md)

## 优化过程

## 1、没有优化
执行SQL语句
```SQL
SELECT u.username, o.order_date, p.name
FROM users u
JOIN orders o ON u.user_id = o.user_id
JOIN products p ON o.product_id = p.product_id
ORDER BY o.order_date DESC;
```

需要4秒钟的时间，EXPLAIN如下:


| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                           | rows    | filtered | Extra                       |
|----|-------------|-------|------------|--------|---------------|---------|---------|-------------------------------|---------|----------|-----------------------------|
|  1 | SIMPLE      | o     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                          | 9742689 |   100.00 | Using where; Using filesort |
|  1 | SIMPLE      | u     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | optimize_train_2.o.user_id    |       1 |   100.00 | NULL                        |
|  1 | SIMPLE      | p     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | optimize_train_2.o.product_id |       1 |   100.00 | NULL                        |

Extra里提示了"Using filesort"，初步推断是ORDER BY导致。

## 2、第一次优化
给orders表的order_date列加上索引，仅需要0.003秒，优化成功。EXPLAIN如下：

| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                           | rows    | filtered | Extra                       |
|----|-------------|-------|------------|--------|---------------|---------|---------|-------------------------------|---------|----------|-----------------------------|
|  1 | SIMPLE      | o     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                          | 9742689 |   100.00 | Using where; Using filesort |
|  1 | SIMPLE      | u     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | optimize_train_2.o.user_id    |       1 |   100.00 | NULL                        |
|  1 | SIMPLE      | p     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | optimize_train_2.o.product_id |       1 |   100.00 | NULL                        |

但是Extra依然提示"Using filesort"，所以说EXPLAIN有时候也不是完全对的。
