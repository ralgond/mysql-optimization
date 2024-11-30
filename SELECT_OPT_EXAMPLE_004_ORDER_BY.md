# 关于ORDER BY和深分页

## 创建数据库

[创建脚本](CREATE_DB.md)

## 优化过程

### 1、没有优化
执行SQL语句
```SQL
SELECT * FROM orders ORDER BY order_date LIMIT 5000000,10;
```

需要3秒钟的时间，EXPLAIN如下:

| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra          |
|----|-------------|--------|------------|------|---------------|------|---------|------|---------|----------|----------------|
|  1 | SIMPLE      | orders | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 9742689 |   100.00 | Using filesort |

### 2、第一次优化

上面可以看见Extra显示Using filesort，可以尝试给order_date添加索引。

再次执行，还是需要3秒。EXPLAIN如下：

| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra          |
|----|-------------|--------|------------|------|---------------|------|---------|------|---------|----------|----------------|
|  1 | SIMPLE      | orders | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 9742689 |   100.00 | Using filesort |

type还是ALL，这说明MYSQL优化器认为走索引回表还不如全表扫描快，所以选择了ALL。

### 3、第二次优化

改写SQL语句为如下：
```SQL
SELECT * FROM orders where order_date >= (SELECT order_date FROM orders ORDER BY order_date LIMIT 5000000,1) ORDER BY order_date LIMIT 10;
```

这次执行只需要0.3秒。EXPLAIN如下：

| id | select_type | table  | partitions | type  | possible_keys  | key            | key_len | ref  | rows    | filtered | Extra       |
|----|-------------|--------|------------|-------|----------------|----------------|---------|------|---------|----------|-------------|
|  1 | PRIMARY     | orders | NULL       | range | idx_order_date | idx_order_date | 3       | NULL | 4871344 |   100.00 | Using where |
|  2 | SUBQUERY    | orders | NULL       | index | NULL           | idx_order_date | 3       | NULL | 5000001 |   100.00 | Using index |

子查询走了覆盖索引，没有回表，索引type为index，父查询走的是范围查询，也避免了全表查询。
