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

