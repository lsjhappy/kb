# Oracle 命令速查

> 全部只读。用**监控账号**跑，别用 sysdba。

## 连库

```bash
sqlplus 用户名/密码@//主机:1521/服务名
```

进库后先设置好显示格式：

```sql
SET LINESIZE 200
SET PAGESIZE 100
SET NUMWIDTH 12
```

## 看表空间（最常查）

```sql
SELECT tablespace_name,
       ROUND(SUM(bytes)/1024/1024/1024, 2) AS 总GB
  FROM dba_data_files
 GROUP BY tablespace_name;

SELECT tablespace_name,
       ROUND(SUM(bytes)/1024/1024/1024, 2) AS 空闲GB
  FROM dba_free_space
 GROUP BY tablespace_name;
```

## 看会话和锁

```sql
-- 当前会话
SELECT sid, serial#, username, status, machine, program
  FROM v$session
 WHERE username IS NOT NULL;

-- 有没有阻塞
SELECT blocking_session, sid, serial#, event, seconds_in_wait
  FROM v$session
 WHERE blocking_session IS NOT NULL;

-- 锁等待
SELECT * FROM v$lock WHERE block > 0;
```

## 看慢 SQL

```sql
SELECT sql_id, elapsed_time/1000000 AS 秒, executions, sql_text
  FROM v$sql
 ORDER BY elapsed_time DESC
 FETCH FIRST 20 ROWS ONLY;
```

## 看进程数（ORA-00020 报错时查）

```sql
SELECT resource_name, current_utilization, limit_value
  FROM v$resource_limit
 WHERE resource_name IN ('sessions','processes');
```

## 找表

```sql
-- 按名字模糊找表
SELECT owner, table_name FROM all_tables
 WHERE table_name LIKE '%RULE%' ORDER BY owner, table_name;

-- 看表结构
SELECT column_name, data_type, data_length
  FROM all_tab_columns
 WHERE table_name = '表名大写' ORDER BY column_id;
```

## ⚠️ 纪律

- **只读**：只用 `SELECT`。要 `UPDATE/DELETE` 先找研发确认。
- **别查全表**：大表加 `WHERE` 和 `FETCH FIRST`。
- **别在业务高峰跑重查询**：会拖慢业务。
