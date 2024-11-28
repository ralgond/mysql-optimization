# 关于EXPLAIN结果中的type字段

## 生成数据

下载数据：

http://downloads.mysql.com/docs/sakila-db.zip

导入数据：

sudo mysql < sakila-schema.sql

sudo mysql < sakila-data.sql

## EXPLAIN的type字段

### ALL
EXPLAIN SELECT * FROM film WHERE rating > 9;

结果如下：
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
|----|-------------|-------|------------|------|---------------|------|---------|------|------|----------|-------------|
|  1 | SIMPLE      | film  | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1000 |    33.33 | Using where |

语句中没有使用任何索引，所以是全表扫描，类型为ALL。

### index
EXPLAIN SELECT title FROM film;

结果如下：
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
|----|-------------|-------|------------|-------|---------------|-----------|---------|------|------|----------|-------------|
|  1 | SIMPLE      | film  | NULL       | index | NULL          | idx_title | 514     | NULL | 1000 |   100.00 | Using index |

语句中的title字段是在film表中建立了索引，所以从索引读title不仅保证有序，而且还比从film表中读要更省IO操作，所以类型为index。

### range
explain select * from payment where customer_id >=300 and customer_id <= 350;

结果如下：
| id | select_type | table   | partitions | type  | possible_keys      | key                | key_len | ref  | rows | filtered | Extra                 |
|----|-------------|---------|------------|-------|--------------------|--------------------|---------|------|------|----------|-----------------------|
|  1 | SIMPLE      | payment | NULL       | range | idx_fk_customer_id | idx_fk_customer_id | 2       | NULL | 1350 |   100.00 | Using index condition |

范围查询就是range类型。

### ref
使用非唯一索引扫描或唯一索引的前缀扫描，返回匹配某个单独值的记录行，例如：

explain SELECT * from payment WHERE customer_id = 350;

结果如下：
| id | select_type | table   | partitions | type | possible_keys      | key                | key_len | ref   | rows | filtered | Extra |
|----|-------------|---------|------------|------|--------------------|--------------------|---------|-------|------|----------|-------|
|  1 | SIMPLE      | payment | NULL       | ref  | idx_fk_customer_id | idx_fk_customer_id | 2       | const |   23 |   100.00 | NULL  |

索引idx_fk_customer_id是非唯一索引，查询条件为等值查询条件customer_id=350，所以扫描索引的类型为ref。

ref还经常出现在join操作中：

explain select b.\*, a.\* from payment a, customer b where a.customer_id =b.customer_id;

结果如下：
| id | select_type | table | partitions | type | possible_keys      | key                | key_len | ref                  | rows | filtered | Extra |
|----|-------------|-------|------------|------|--------------------|--------------------|---------|----------------------|------|----------|-------|
|  1 | SIMPLE      | b     | NULL       | ALL  | PRIMARY            | NULL               | NULL    | NULL                 |  599 |   100.00 | NULL  |
|  1 | SIMPLE      | a     | NULL       | ref  | idx_fk_customer_id | idx_fk_customer_id | 2       | sakila.b.customer_id |   26 |   100.00 | NULL  |

### ref_eq
类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配；简单来说，就是多表连接中使用 primary key或者 unique index作为关联条件。

explain select * from film a, film_text b where a.film_id = b.film_id;

结果如下：
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref              | rows | filtered | Extra |
|----|-------------|-------|------------|--------|---------------|---------|---------|------------------|------|----------|-------|
|  1 | SIMPLE      | b     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL             | 1000 |   100.00 | NULL  |
|  1 | SIMPLE      | a     | NULL       | eq_ref | PRIMARY       | PRIMARY | 2       | sakila.b.film_id |    1 |   100.00 | NULL  |

### const/system
单表中最多有一个匹配行，查询起来非常迅速，所以这个匹配行中的其他列的值可以被优化器在当前查询中当作常量来处理，例如，根据主键 primary key或者唯一索引unique index进行的查询。

explain select * from film where film_id=1;

结果如下：
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
|----|-------------|-------|------------|-------|---------------|---------|---------|-------|------|----------|-------|
|  1 | SIMPLE      | film  | NULL       | const | PRIMARY       | PRIMARY | 2       | const |    1 |   100.00 | NULL  |
