### 1. RDS 备份

- RDS 支持自动备份
- 实时捕获事务日志
- 默认情况下启用，保留期为7天（0-35天保留期，0=禁用自动备份）
- 您可以提供备份窗口时间和备份保留天数
- 第一个备份是完整备份，后续备份是增量备份
- 数据存储在 S3 存储桶中（由 RDS 服务拥有和管理，您不会在 S3 控制台中看到它们）
- 建议使用 Multi-AZ 选项来避免备份运行时的性能问题
- 与 AWS Backup 服务集成以实现集中管理
- 支持 PITR，而快照则不支持
- 复制备份会变为快照，可以跨越账号、region
- Point-In-Time Recovery 时间间隔为 5 分钟

### 2. 从快照还原

- 只能恢复到新实例
- 一个实例可以有一个或多个数据库，所有这些数据库都将被恢复
- 要保留相同的名称，请先删除或重命名现有实例
- 无法直接从共享和加密的快照中恢复（先复制，然后从副本中恢复）
- 无法直接从另一个区域恢复（先复制，然后从副本恢复）
- 可以从 VPC 外的数据库实例快照恢复到 VPC 内（但相反则不行）
- 默认情况下，还原的集群使用：
  - 新建安全组
  - 默认参数组
  - 与快照关联的选项组
- 从快照恢复时，请确保
  - 选择正确的安全组以确保恢复的数据库的连接
  - 为还原的数据库选择正确的参数组
  - 建议保留快照的参数组，以帮助使用正确的参数组进行恢复

### 3. 导出快照到 S3

- 可以导出所有类型的备份（自动/手动或使用 AWS 备份服务创建的备份）
- 如何出口？
  - 设置具有适当 IAM 权限的 S3 存储桶，并为 SSE 创建KMS密钥
  - 使用控制台（Actions -> Export to Amazon S3）或使用 start Export task CLI 命令导出快照
- 导出在后台运行
- 不会影响数据库性能
- 以 Apache Parquet 格式导出的数据（压缩、一致）
- 允许您使用 Athena 或 Redshift Spectrum 分析数据库数据

### 4. 比较 RDS DR 策略

|                   | RTO<br />Recovery Time Objective | RPO<br />Recovery Point Object |  Cost  |     Scope     |
| ----------------- | :------------------------------: | :----------------------------: | :----: | :-----------: |
| Automated backups |               Good               |             Better             |  Low   | Single Region |
| Manual snapshots  |              Better              |              Good              | Medium | Cross-Region  |
| Read replicas     |               Best               |              Best              |  High  | Cross-Region  |

### 5. 如何解决复制错误的建议

- 调整副本的大小以匹配源数据库（存储大小和数据库实例类）
- 对源数据库和副本使用兼容的数据库参数组设置
- 例如，读取副本允许的最大数据包必须与源数据库实例的数据包相同
- 监视副本实例的 Replication State 字段
- 如果 Replication State = Error，然后查看 Replication Error 字段中的错误详细信息
- 使用 RDS 事件通知获取有关此类副本问题的警报
- 写入读取复制副本上的表
  - 将只读设置为0以使读取副本可写
  - 仅用于维护任务（如仅在复制副本上创建索引）
  - 如果您在读取副本上写入表，可能会使其与源数据库不兼容并破坏复制
  - 因此，在完成维护任务后立即设置`read-only=1`
- 只有像lnnoDB这样的事务存储引擎才支持复制，使用MylSAM这样的引擎会导致复制错误
- 使用不安全的非确定性查询（如SYSDATE）（可能会破坏复制）
- 您可以跳过复制错误（如果不是主要错误），也可以删除并重新创建复制副本

对于MySQL：

- 错误或数据不一致b/w源实例和replica
  - 可能是由于 binlog 事件或 lnnoDB 重做日志在 replica 或源实例失败期间未刷新而发生的
  - 必须手动删除并重新创建复制
- 预防性建议：
  - sync_binlog=1
  - innodb_flush_log_at_trx_commit=1
  - innodb_support_xa=1
- 这些设置可能会降低性能（因此在转到生产前进行测试）

### 6. Aurora Replicas vs MySQL Replicas

| Feature                                          | Amazon Aurora Replicas      | MySQL Replicas                         |
| ------------------------------------------------ | --------------------------- | -------------------------------------- |
| Number of replicas                               | Up to 15                    | Up to 5                                |
| Replication type                                 | Asynchronous (milliseconds) | Asynchronous (seconds)                 |
| Performance impact on primary                    | Low                         | High                                   |
| Replica location                                 | In-region                   | Cross-region                           |
| Act as failover target                           | Yes (no data loss)          | Yes (potentially minutes of data loss) |
| Automated failover                               | Yes                         | No                                     |
| Support for user-defined replication delay       | No                          | Yes                                    |
| Support for different data or schema vs. primary | No                          | Yes                                    |

### 7. Comparison of RDS Deployments

<table>
<thead>
  <tr>
    <th></th>
    <th>Read replicas</th>
    <th>Multi-AZ deployments (Single-Region)</th>
    <th>Multi-Region deployments</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Main purpose</td>
    <td>Scalability</td>
    <td>HA</td>
    <td>DR and performance</td>
  </tr>
  <tr>
    <td rowspan="2">Replication method</td>
    <td rowspan="2">Asynchronous (eventual consistency)</td>
    <td>Synchronous</td>
    <td rowspan="2">Asynchronous (eventual consistency)</td>
  </tr>
  <tr>
    <td>Asynchronous (Aurora)</td>
  </tr>
  <tr>
    <td>Accessibility</td>
    <td>All replicas can be used for read scaling</td>
    <td>Active-Passive (standby not accessible)</td>
    <td>All regions can be used for reads</td>
  </tr>
  <tr>
    <td rowspan="2">Automated backups</td>
    <td rowspan="2">No backups configured by default</td>
    <td>Taken from standby</td>
    <td rowspan="2">Can be taken in each region</td>
  </tr>
  <tr>
    <td>Taken from shared storage layer (Aurora)</td>
  </tr>
  <tr>
    <td>Instance placement</td>
    <td>Within-AZ, Cross-AZ, or Cross-Region</td>
    <td>At least two AZs within region</td>
    <td>Each region can have a Multi-AZ deployment</td>
  </tr>
  <tr>
    <td rowspan="2">Upgrades</td>
    <td>Independent from source instance</td>
    <td>On primary</td>
    <td>Independent in each region</td>
  </tr>
  <tr>
    <td colspan="3">All instances together (Aurora) </td>
  </tr>
  <tr>
    <td rowspan="2">DR (Disaster Recovery)</td>
    <td>Can be manually promoted to a standalone instance</td>
    <td>Automatic failover to standby</td>
    <td rowspan="2">Aurora allows promotion of a secondary region to be the master</td>
  </tr>
  <tr>
    <td>Can be promoted to primary (Aurora)</td>
    <td>Automatic failover to read replica (Aurora)</td>
  </tr>
</tbody>
</table>


### 8. DynamoDB Terminology Compared to SQL

| SQL / RDBMS                | DynamoDB                                                     |
| -------------------------- | ------------------------------------------------------------ |
| Tables                     | Tables                                                       |
| Rows                       | Items                                                        |
| Columns                    | Attributes                                                   |
| Primary Keys - Multicolumn | Primary Keys - Mandatory, minimum one and maximum two attributes |
| Indexes                    | Local Secondary Indexes                                      |
| Views                      | Global Secondary Indexes                                     |

### 9. AWS 数据库之间的比较

| Database         | Type of data                   | Workload                                           | Size          | Performance                                                  | Operational overhead |
| ---------------- | ------------------------------ | -------------------------------------------------- | ------------- | ------------------------------------------------------------ | -------------------- |
| RDS DBs          | Structured                     | Relational / OLTP / simple OLAP                    | Low TB range  | Mid-to-high throughtput, low latency                         | Moderate             |
| Aurora Mysql     | Structured                     | Relational / OLTP / simple OLAP                    | Mid TB range  | High throughtput, low latency                                | Low-to-moderate      |
| Aurora PostgrSQL | Structured                     | Relational / OLTP / simple OLAP                    | Mid TB range  | High throughtput, low latency                                | Low-to-moderate      |
| Redshift         | Structured / Semi-structured   | Relational / OLAP / DW                             | PB range      | Mid-to-high latency                                          | Moderate             |
| DynamoDB         | Semi-structured                | Non-relational / Key-Value / OLTP / Document store | High TB range | Ultra-high throughtput, low latency, ultra-low latency with DAX | Low                  |
| ElastiCache      | Semi-structured / Unstructured | Non-relational / In-memory caching / Key-Value     | Low TB range  | High throughtput, ultra-low latency                          | Low                  |
| Neptune          | Graph data                     | Highly connected graph datasets                    | Mid TB range  | High throughtput, ultra-low latency                          | Low                  |
