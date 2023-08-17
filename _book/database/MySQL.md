### 1. Explain 语句结果中各个字段分别表示什么
|     列名      | 描述                                                         |
| :-----------: | ------------------------------------------------------------ |
|      id       | 查询语句中每出现一个 SELECT 关键字，MySQL 就会为它分配一个唯一的 id 值，某些子查询会被优化为 join 查询，那么出现的 id 会一样 |
|  select_type  | SELECT 关键字对应的那个查询的类型                            |
|     table     | 表名                                                         |
|  partitions   | 匹配的分区信息                                               |
|     type      | 针对单表的查询方式(全表扫描、索)                             |
| possible_keys | 可能用到的索引                                               |
|      key      | 实际上使用的索引                                             |
|    key_len    | 实际使用到的索引长度                                         |
|      ref      | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息       |
|     rows      | 预估的需要读取的记录条数                                     |
|   filtered    | 某个表经过搜索条件过滤后剩余记录条数的百分比                 |
|     Extra     | 一些额外的信息，比如排序等                                   |

### 2. Innodb 是如何实现事务的

Innodb 通过 Buffer Pool，LogBuffer，Redo Log，Undo Log 来实现事务，以一个 update 语句为例:

- Innodb 在收到一个update语句后，会先根据条件找到数据所在的页，并将该页缓存在 Buffer Pool 中
- 执行 update 语句，修改 Buffer Pool 中的数据，也就是内存中的数据
- 针对 update 语句生成一个 RedoLog 对象，并存入 LogBuffer 中
- 针对 update 语句生成 undolog 日志，用于事务回滚
- 如果事务提交，那么则把 Redolog 对象进行持久化，后续还有其他机制将 Buffer Pool 中所修改的数据页持久化到磁盘中
- 如果事务回滚，则利用 undolog 日志进行回滚

### 3. 零时刻导致 Flyway 执行失败

运行时，历史脚本中的 Flyway 失败，因为带有时间戳的脚本没有默认值。检查本地 MySQL 环境的 sql_mode，删除`NO_ZERO_DATE`和`NO_ZERO_IN_DATE`。

```sql
SELECT @@GLOBAL.sql_mode;
--- SET时移除NO_ZERO_DATE和NO_ZERO_IN_DATE
SET GLOBAL sql_mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION";
```

