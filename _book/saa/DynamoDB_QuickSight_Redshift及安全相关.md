### 1. DynamoDB

- 读取容量单位Readcapacityunit(RCU)：从表中读取数据的每个API调用都是一个读取请求。读取请求可以是强一致性、最终一致性或事务性读取请求。对最大4KB的项目，一个RCU每秒可以执行一个强一致性读取请求。大于4KB的项目需要额外的RCU。对最大4KB的项目，一个RCU每秒可以执行两个最终一致性读取请求。对最大4KB的项目，事务性读取请求需要两个RCU才能每秒执行一个读取请求。例如，对8KB项目的强一致性读取需要两个RCU，对8KB项目的最终一致性读取需要一个RCU，而对8KB项目的事务性读取需要四个RCU。有关详细信息，请参阅读取一致性。
- 写入容量单位Writecapacityunit(WCU)：将数据写入表的每个API调用都是一个写入请求。对最大1KB的项目，一个WCU每秒可以执行一个标准写入请求。大于1KB的项目需要额外的WCU。对最大1KB的项目，事务性写入请求需要两个WCU才能每秒执行一个写入请求。例如，对1KB项目的标准写入请求需要一个WCU，对3KB项目的标准写入请求需要三个WCU，而对3KB项目的事务性写入请求需要六个RCU。

### 2. QuickSight
Amazon QuickSight 允许您的企业中的所有人通过使用自然语言提问以了解您的数据，通过交互式控制面板探索，或自动查找由机器学习支持的模式和异常值。

### 3. Redshift

- Redshift is based on PostgreSQL, but it’s not used for OLTP
- It's OLAP – online analytical processing (analytics and data warehousing)
- 10x better performance than other data warehouses, scale to PBs of data
- Columnar storage of data (instead of row based)
- Massively Parallel Query Execution (MPP)
- Pay as you go based on the instances provisioned
- Has a SQL interface for performing the queries
- BI tools such as AWS Quicksight or Tableau integrate with it

###  4. WAF, Shield Advanced

[什么是AWS WAF、AWS和 ShieldAWS Firewall Manager?](https://docs.aws.amazon.com/zh_cn/waf/latest/developerguide/what-is-aws-waf.html)

### 5. CloudHSM
AWS CloudHSM 是基于云的硬件安全模块 (HSM)，让您能够在 AWS 云上轻松生成和使用自己的加密密钥。借助 CloudHSM，您可以使用经过 FIPS 140-2 第 3 级验证的 HSM 管理自己的加密密钥。CloudHSM 让您可以灵活选择使用行业标准的 API 与应用程序集成，这些 API 包括 PKCS#11、Java 加密扩展 (JCE) 和 Microsoft CryptoNG (CNG) 库等。

此外，CloudHSM 符合标准，让您可以将所有密钥导出到大多数其他商用 HSM，具体取决于您的配置。它是一项完全托管的服务，可为您自动执行耗时的管理任务，例如硬件预置、软件修补、高可用性和备份。借助 CloudHSM，您还能够通过按需添加和删除 HSM 容量进行快速扩展和收缩，无任何预付费用。

### 6. Secrets Manager
AWS Secrets Manager 可帮助您保护访问应用程序、服务和 IT 资源所需的密钥。该服务使您能够轻松地跨整个生命周期轮换、管理和检索数据库凭证、API 密钥和其他密钥。用户和应用程序通过调用 Secrets Manager API 来检索密钥，无需对纯文本的敏感信息进行硬编码。Secrets Manager 通过适用于 Amazon RDS、Amazon Redshift 和 Amazon DocumentDB 的内置集成实现密钥轮换。此外，该服务还可以扩展到其他类型的密钥，包括 API 密钥和 OAuth 令牌。此外，Secrets Manager 使您能够使用精细权限来访问机密信息，并集中审核对 AWS 云、第三方服务和本地资源的密钥轮换。