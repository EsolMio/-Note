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