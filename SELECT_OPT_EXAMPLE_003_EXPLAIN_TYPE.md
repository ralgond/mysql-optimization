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


