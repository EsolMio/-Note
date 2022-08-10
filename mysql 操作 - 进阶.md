# Mysql优化/进阶操作

> [JavaGuide MySQL 操作集合](https://github.com/Snailclimb/JavaGuide/blob/main/docs/database/mysql/a-thousand-lines-of-mysql-study-notes.md)

## 记录

- 表：select nformation_schema.tables，搜索：table_name = 表名

- 列columns：columns，搜索：列名/属性

- 执行列表（事务）：show processlist

- metadata lock（事务）：preformance_schema.metadata_locks，搜索：object_name = 表名

- set profiling = 1 跟踪后续执行过程
    - 执行语句ddl/dml（开头添加 explain，分析sql执行）
    - select * from information_schema.innodb_trx\G processlist相关信息
    - show profile cpu,block io for query 3; profile信息
- show master status 在主数据库中显示主主数据库信息

- create user 'slave'@'%' identified by '123456'; grant replication slave, replication client on *.* to 'slave'@'%' 在主数据库中创建数据同步用户

- change master to ... 从数据库中配置主从复制
    - master_host: 主数据库主机号/ip
    - master_port: 主端口
    - master_user: 主数据库中创建的同步用户名
    - master_log_file: 指定主数据库中binlog的文件名
    - master_log_pos: 指定从主数据库复制起始位置
    - master_connect_retry: 断联时重连时间间隔

- show slave status \G 在从数据库查看主从同步状态。Slave_IO_Running: No Slave_SQL_Running: No（未开启主从同步）

- start slave 在从数据库中开启主从同步
  
### 查看sql执行计划

- `explain sql语句`

### 跟踪优化器

```sql
-- 开启optimizer_trace
mysql> set session optimizer_trace="enabled=on", end_markers_in_json=on;

mysql> select ...;

-- 查看跟踪结果
mysql> select trace from information_schema.OPTIMIZER_TRACE\G;
```

### 强制开启index（优化器误判问题）- 表：test.large_item

1. 经过优化器选择后未使用索引，预期搜索行数为`187222`

```sql
MariaDB [test]> explain select id, score, name from large_item where score >= 15510 and name like '%name';
+------+-------------+------------+------+---------------+------+---------+------+--------+-------------+
| id   | select_type | table      | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+------+-------------+------------+------+---------------+------+---------+------+--------+-------------+
|    1 | SIMPLE      | large_item | ALL  | score_name    | NULL | NULL    | NULL | 187222 | Using where |
+------+-------------+------------+------+---------------+------+---------+------+--------+-------------+
```

2. 强制使用索引`force index(score_name)`，预期索引数为`93611`
```sql
MariaDB [test]> explain select id, score, name from large_item force index(score_name) where score >= 10 and name like '%name';
+------+-------------+------------+-------+---------------+------------+---------+------+-------+------------------------------------+
| id   | select_type | table      | type  | possible_keys | key        | key_len | ref  | rows  | Extra
             |
+------+-------------+------------+-------+---------------+------------+---------+------+-------+------------------------------------+
|    1 | SIMPLE      | large_item | range | score_name    | score_name | 4       | NULL | 93611 | Using index condition; Using where |
+------+-------------+------------+-------+---------------+------------+---------+------+-------+------------------------------------+
```

**特殊情况**: 并非使用index搜索就比全表快，比如 `select name from test.large_item where score >= 10;` 比使用 `force index(score_name)` 快（0.33s，1.09s）

- 原因：索引`score_name`中，为控制索引key长度`name`使用了前缀索引，`force index`后还需要回表查询完整数据