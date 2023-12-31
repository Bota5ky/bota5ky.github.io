### 1. SQL Server Compact Edition

**SQL CE 中 sp_rename 仅支持表的修改**

```sql
sp_rename 'oldTableName','newTableName';
```

在 SQL Server 2005 Management Studio 中，您必须使用新名称创建一个新列，然后使用旧列中的值更新它，然后删除旧列。如果列是索引的一部分，那么最后一个操作是困难的。

**SQL CE 查询表信息**

```sql
SELECT table_name_, column_name FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name_='stu';
```

**SQL CE 和其他 SQL 的区别**

- [SQL CE 3.5 和其他 SQL 的区别](https://docs.microsoft.com/zh-cn/previous-versions/sql/compact/sql-server-compact-3.5/bb896140%28v=sql.100%29)
- [SQL CE 3.5 SP2和其他 SQL 的区别](https://docs.microsoft.com/zh-cn/previous-versions/sql/compact/sql-server-compact-3.5-sp2/bb896140%28v=sql.105%29)
- [SQL CE 4.0和其他 SQL 的区别](https://docs.microsoft.com/zh-cn/previous-versions/sql/compact/sql-server-compact-4.0/bb896140%28v=sql.110%29)

**SQL CE 使用 EFCore 连接并持久化对象：**

https://entityframework-extensions.net/efcore-sql-server-compact-provider

**确定 Firebird SQL 版本**

   ```sql
SELECT rdb$get_context('SYSTEM', 'ENGINE_VERSION') as version from rdb$database;
   ```

### 2. 不同数据库之间的类型映射关系

- 常用关键字翻译

| T-SQL                                                        | PostgreSQL                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| cast(`value` AS datetime)<br>convert(datetime, `value`)      | cast(`value` AS timestamp(3))                                |
| datepart(day, `value`)                                       | extract(day FROM `value`)                                    |
| dateadd(day, `value1`, `value2`)                             | `value2` + `value1` * INTERVAL '1 day'                       |
| dateadd(d,-1,dateadd(mm, datediff(m,0,`column_name`)+1,0))<br>当月的最后一天 | date_trunc('month', `column_name`) + INTERVAL '1 month- 1 day' |
| len(`value`)                                                 | char_length(`value`)                                         |
| +                                                            | &#124;&#124;                                                 |
| IDENTITY (1,1)                                               | GENERATED BY DEFAULT AS IDENTITY                             |
| ROWVERSION                                                   | bytea                                                        |
| CREATE TABLE `table_name` (`column_name` varbinary(46) NOT NULL DEFAULT ((0))) | CREATE TABLE `table_name` (`column_name` bytea NOT NULL DEFAULT E'\\x00000000') |
| DECLARE `@tablename` TABLE(`column_name1` nvarchar(500), `column_name2` nvarchar(500)) | CREATE TEMPORARY TABLE `tablename` (`column_name1` nvarchar(500), `column_name2` nvarchar(500))<br>WITH `tablename` AS (SELECT …) |
| NOCHECK CONSTRAINT all<br>WITH CHECK CHECK CONSTRAINT all    | DISABLE TRIGGER ALL<br>ENABLE TRIGGER ALL                    |

- 常用类型映射

| T-SQL                          | PostgreSQL                                                   |
| :----------------------------- | :----------------------------------------------------------- |
| smallint                       | smallint, int2                                               |
| int                            | integer, int, int4                                           |
| bigint                         | bigint, int8                                                 |
| tinyint                        | **不支持**                                                   |
| float(n)  `1 <= n <= 24`, real | float(n)  `1 <= n <= 24`, real, float4                       |
| float(n)  `25 <= n <= 53`      | double precision, float(n)  `25 <= n <= 53`, float8          |
| numeric, decimal               | numeric, decimal                                             |
| money, smallmoney              | money<br>In SQL Server, money is `(19,4)` and smallmoney is `(10,4)` in , but  in Postgres money is `(19,2)` |
| varbinary(n)                   | bytea with check<br />PostgreSQL uses 'check' to simulate `n` |
| varbinary(max), image          | bytea                                                        |
| binary(n)                      | **不支持**                                                   |
| date                           | date                                                         |
| datetime                       | timestamp(3) without time zone                               |
| datetime2(n)                   | timestamp(n) without time zone<br>In SQL Server `0 <= n <= 7`, but in PostgreSQL `0 <= n <=6` |
| datetimeoffset(n)              | timestamp(n) with time zone, timestamptz<br>In SQL Server `0 <= n <= 7`, but in PostgreSQ `0 <= n <=6` |
| bit                            | bit, bit(1), boolean, bool                                   |
| char(n)                        | character(n), char(n)                                        |
| nchar(n)                       | character(n), char(n)<br>For UCS-2 encoding, the storage size is two times n bytes |
| varchar(n)                     | character varying(n), varchar(n)                             |
| nvarchar(n)                    | character varying(n), varchar(n)<br>For UCS-2 encoding, the storage size is two times n bytes |
| text                           | text                                                         |
| ntext                          | text                                                         |
