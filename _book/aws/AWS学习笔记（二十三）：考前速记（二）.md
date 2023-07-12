- Inspector 漏洞修复

- GuardDuty 威胁检测

- WAF(Web Application Firewall) 跨站脚本、SQL注入、定义IP规则 ALB

- Sheild DDos NLB

- CloudHSM(Hardware Security Module) 生成和使用密钥

- Secrets Manager 保护、轮换密钥、凭证

- Parameter Store 不支持密码轮换

- KMS(Key Management Service) 创建和管理加密密钥，并控制其在各种 AWS 服务和应用程序中的使用，支持轮换在HSM中生成的密钥

- Secrets Manager是管理/储存secret的地方；KMS是管理加密金钥(key)的地方。而Secrets Manager新增secrets时需要将secret加密(encrypt)，后续要使用secret时需要解密(decrypt)，而加解密secrets使用的key则是由KMS来管理。所以Secrets Manager与KMS经常搭配使用。

- DataSync 本地 → AWS，不应该长时间持续使用

- Direct Connect 建立需要约几个月

- 支持REST API：S3、API Gateway

- 支持VPC Gateway：S3、DynamoDB

- CloudTrail 默认开启管理事件

- DynamoDB DAX 内存缓存，微秒级 → Redis 亚毫秒级

- Fsx 支持SMB(Server Message Block)、ACL、DFSR(Distributed File System Replication 分布式文件系统复制)

- FSx for Lustre 亚毫秒级延迟、HPC(FSx for Windows 不支持)

- CloudFront 不在子网内，不适用ACL

- Auto Scaling在最后一个启动的实例完成冷却期后才会接受扩展活动请求

- Redshift 随着数据和查询复杂度的增长，保持高性能；防止报告和分析处理干扰OLTP(Online Transaction Processing)工作负载的性能；使用标准SQL和现有BI工具保存和查询大量结构化数据；自动运行自己的数据仓库和处理设置、持久性、监控、扩展和修补

- Redshift 使用列式数据存储

- Gateway-cached 将数据存储在Amazon Simple Storage Service (Amazon S3) 并在本地保留经常访问的数据子集的副本

- Gateway-stored 可以将所有数据储存在本地存储，然后异步备份将此数据传输到 Amazon S3

- Memcached不支持数据复制或高可用性，Redis支持

- EBS snapshot 最大1TB

- 一个 VPC 可以跨越多个可用区，子网必须驻留在单个可用区内

- SES(Simple Email Service)

- Oracle Data Pump导入大量数据；Oracle SQL Developer 导入20MB数据库

- EBS optimimzed instance 平均IOPS 90%

- AWS 导入/导出支持：导入导出S3；导入EBS；导入Glacier。不支持：导出EBS；导出Glacier

- Route 53原生支持带有内部健康检查的ELB，开启评估目标健康，关闭关联Health Check

- 每个Region限制5个弹性IP

- EC2-Classic须使用专门的安全组，不能用EIP

- LB使用HTTP检查实例的运行状况

- Amazon EMR(Elastic MapReduce) 使用 Apache Hadoop 作为其分布式数据处理引擎。Hadoop 是一个开源的 Java 软件框架，支持在大型商品硬件集群上运行的数据密集型分布式应用程序。Hadoop 实现了一个名为“MapReduce”的编程模型，其中数据被分成许多小的工作片段，每个片段都可以在集群中的任何节点上执行。该框架已被开发人员、企业和初创公司广泛使用，并被证明是处理高达 PB 级数据的可靠软件平台，数千台商品机器集群的数据。

- ECS目前仅支持Docker容器类型

- EBS gp2 16000 IOPS

- ACL按编号从低到高的顺序处理

- ALB不支持静态IP地址，而NLB支持静态IP地址。要将静态IP分配给ALB，有两种解决方案。1. 在 ALB 前面使用NLB和lambda 2.使用全球加速器

- Storage Gateway支持NFS、SMB、Active Directory

- S3根据文件在S3中的天数移动，EFS根据文件闲置的天数移动

- Parameter Store 是 AWS Systems Manager 的一项功能，可为配置数据管理和机密管理提供安全的分层存储。您可以将密码、数据库字符串、Amazon 系统映像 (AMI) ID 和许可证代码等数据存储为参数值。您可以将值存储为纯文本或加密数据。您可以使用创建参数时指定的唯一名称在脚本、命令、SSM 文档以及配置和自动化工作流中引用 Systems Manager 参数。

- 无法禁用VPC和子网的IPv4支持

- 在ACM（AWS Certificate Manager）导入SSL证书

- Kinesis Data Streams只保留7天数据

- Brusting EFS随着标准存储类中文件系统大小的增长而扩展

- 仅出口IPv6 流量请使用Internet 网关，要通过IPv4出站请改用 NAT 网关

- 在ALB上安装了SSL证书，是免费的。ALB从网上获取https，转换为http，然后发送到EC2

- Lambda只能使用基于轮询的调用而不是通过异步调用来调用SQS

- ES（Elasticearch Service）做数据可视化，不是查询数据

- Lamdba最大内存10GB

- Alias根域名，CNAME不行

- Fargate不支持Fsx

- 只有IAM角色可用于访问跨账户资源

- DAX -> DynamoDB的缓存，即使在每秒数百万个请求的情况下，也可将性能提高多达 10 倍（从毫秒到微秒）

- AWS KMS删除强制执行等待期7-30天

- SSE-S3、SSE-C不提供审计跟踪加密密钥使用的能力

- FIFO 队列支持每秒最多300个操作（每个操作最多批处理 10 条消息）

- GuardDuty 支持的数据源：VPC 流日志、DNS 日志、CloudTrail 事件

- API Gateway可以使用令牌桶算法限制对您的 API 的请求

- AWS Managed Microsoft AD 不支持 Microsoft 的分布式文件系统 (DFS)

- S3 Standard 上的测试文件存储成本 < EFS 上的测试文件存储成本0.3 < EBS 上的测试文件存储成本0.1

- Firehose 无法直接写入 DynamoDB 表

- Lambda支持C#/.NET、Go、Java、Node.js、Python、Ruby

- Lambda目前支持每个区域每个 AWS 账户1000 次并发执行

- Neptune图形数据库服务

- Global Accelerator 无助于加快文件传输到 S3 的速度

- Global Accelerator 是一项服务，可提高本地或全球用户的应用程序的可用性和性能。它提供静态 IP 地址，充当单个或多个 AWS 区域中的应用程序终端节点的固定入口点，例如您的应用程序负载均衡器、网络负载均衡器或 Amazon EC2 实例

- RDS多可用区遵循同步复制，Auroa遵循异步复制

- GuardDuty禁用服务将删除所有剩余数据，包括您的发现和配置，然后再放弃服务权限并重置服务

- S3 Glacier Instant Retrieval 提供对归档存储的最快访问，具有与 S3 Standard 和 S3 Standard-IA 存储类相同的吞吐量和毫秒访问。S3 Glacier Instant Retrieval 非常适合需要立即访问的存档数据，例如医学图像、新闻媒体资产或用户生成的内容存档，最低存储期限费用为 90 天

- S3 Standard-IA 和 S3 One Zone-IA 的最低存储期限费用为 30 天

- S3 Standard 没有最低存储持续时间费用和检索费用，S3 Standard-IA 和 S3 One Zone-IA 有检索费用

- 专用主机与专用实例的区别：可见性、放置位置、自带许可，在性能、安全性或物理特性方面没有区别

- AWS Resource Access Manager (RAM) 是一项服务，可让您轻松安全地与任何 AWS 账户或在您的 AWS 组织内共享 AWS 资源。您可以与 RAM 共享 AWS Transit Gateways、子网、AWS License Manager 配置和 Amazon Route 53 Resolver 规则资源。RAM 消除了在多个账户中创建重复资源的需要，从而减少了在您拥有的每个账户中管理这些资源的运营开销。您可以在多账户环境中集中创建资源，并通过三个简单的步骤使用 RAM 跨账户共享这些资源：创建资源共享、指定资源和指定账户。RAM 可供您免费使用。

  正确的解决方案是使用 RAM 在 VPC 内共享子网。这将允许所有 EC2 实例部署在同一个 VPC 中（尽管来自不同的账户）并轻松地相互通信。

- AWS PrivateLink 通过消除数据暴露于公共 Internet 来简化与基于云的应用程序共享的数据的安全性。AWS PrivateLink 在 Amazon 网络上安全地提供 VPC、AWS 服务和本地应用程序之间的私有连接。私人链接是这个问题的干扰因素。Private Link 用于在一个账户中的 NLB 前端的应用程序和另一个账户中的弹性网络接口 (ENI) 之间创建私有连接，无需 VPC 对等互连并允许两者之间的连接保留在内部AWS 网络。

- Amazon Cognito 的两个主要组件是用户池和身份池。身份池提供 AWS 凭证以授予您的用户访问其他 AWS 服务的权限。要使您的用户池中的用户能够访问 AWS 资源，您可以配置一个身份池以将用户池令牌交换为 AWS 凭证。因此，身份池本身并不是一种身份验证机制，因此不是此用例的选择。

- 使用存储在 AWS Key Management Service (SSE-KMS)中的客户主密钥 (CMK) 进行服务器端加密 - 使用存储在 AWS Key Management Service (SSE-KMS) 中的客户主密钥 (CMK) 进行服务器端加密类似于 SSE- S3。SSE-KMS 为您提供审计跟踪，显示您的 CMK 的使用时间和用户。此外，您可以创建和管理客户托管的 CMK 或使用您、您的服务和您所在区域独有的 AWS 托管 CMK。

- 使用 Amazon S3 托管密钥 (SSE-S3)进行服务器端加密 - 当您使用 Amazon S3 托管密钥 (SSE-S3) 进行服务器端加密时，每个对象都使用唯一密钥进行加密。作为额外的保护措施，它使用定期轮换的主密钥对密钥本身进行加密。所以这个选项是不正确的。

- 为私有托管区域启用 DNS 主机名和 DNS 解析- DNS 主机名和 DNS 解析是私有托管区域的必需设置。私有托管区域的 DNS 查询只能由 Amazon 提供的 VPC DNS 服务器解析。因此，必须启用这些选项才能使您的私有托管区域正常工作

- DNS 主机名和 DNS 解析是私有托管区域的必需设置

- SNS FIFO由SQS FIFO订阅，标准SNS由标准SQS订阅

- Kinesis Agent 是一个独立的 Java 软件应用程序，它提供了一种简单的方法来收集数据并将其发送到 Kinesis Data Streams 或 Kinesis Firehose

- Kinesis Agent 只能写入 Kinesis Data Streams，而不能写入 Kinesis Firehose - Kinesis Agent 是一个独立的 Java 软件应用程序，它提供了一种简单的方法来收集数据并将其发送到 Kinesis Data Streams 或 Kinesis Firehose

- Amazon Kinesis Data Firehose 是将流数据可靠地加载到数据湖、数据存储和分析工具中的最简单方法。它是一项完全托管的服务，可自动扩展以匹配您的数据吞吐量，并且无需持续管理。它还可以在加载数据之前对数据进行批处理、压缩、转换和加密，从而最大限度地减少目的地使用的存储量并提高安全性。

- Amazon Redshift 是一种完全托管的 PB 级基于云的数据仓库产品，专为大规模数据集存储和分析而设计。您不能使用 Redshift 从 IoT 源中捕获键值对中的数据

- 在多主集群中，所有数据库实例都可以执行写入操作。当写入器数据库实例不可用时，不会进行任何故障转移，因为另一个写入器数据库实例可立即接管故障实例的工作。AWS 将这种可用性称为连续可用性，以区别于单主集群提供的高可用性（故障转移期间有短暂的停机时间）。

- 启动模板类似于启动配置，因为它指定实例配置信息，例如 Amazon 系统映像 (AMI) 的 ID、实例类型、密钥对、安全组以及您用于启动的其他参数EC2 实例。此外，定义启动模板而不是启动配置允许您拥有多个版本的模板。多类型

- 启动配置是 Auto Scaling 组用来启动 EC2 实例的实例配置模板。创建启动配置时，您需要为实例指定信息，例如 Amazon 系统映像 (AMI) 的 ID、实例类型、密钥对、一个或多个安全组以及块储存设备映射。单类型

- ASG未终止运行状况不佳的 Amazon EC2 实例：实例的运行状况检查宽限期尚未到期、实例可能处于受损状态、实例未通过 ELB 运行状况检查

- AWS 支持 IAM 实体（用户或角色）的权限边界。权限边界是一项高级功能，用于使用托管策略设置基于身份的策略可以授予 IAM 实体的最大权限。实体的权限边界允许它仅执行其基于身份的策略及其权限边界所允许的操作。在这里，我们必须使用 IAM 权限边界。它们只能应用于角色或用户，而不是 IAM 组

- 服务控制策略 (SCP) 是一种可用于管理组织的策略。SCP 提供对组织中所有帐户的最大可用权限的集中控制，使您能够确保您的帐户符合组织的访问控制准则。SCP 仅在启用了所有功能的组织中可用。如果您的组织仅启用了整合计费功能，则 SCP 不可用。将 SCP 附加到 AWS Organizations 实体（根、OU 或账户）定义了委托人可以执行的操作的防护机制。如果您考虑此选项，由于此问题中未提及 AWS Organizations，因此我们无法应用 SCP

- AWS OpsWorks 是一项配置管理服务，可提供 Chef 和 Puppet 的托管实例。Chef 和 Puppet 是自动化平台，允许您使用代码来自动化服务器的配置。OpsWorks 允许您使用 Chef 和 Puppet 来自动化跨 Amazon EC2 实例或本地计算环境配置、部署和管理服务器的方式

| Type of Access Control | AWS Account Level Control | User Level Control |
| ---------------------- | ------------------------- | ------------------ |
| IAM Policies           | No                        | Yes                |
| ACLs                   | Yes                       | No                 |
| Bucket Policies        | Yes                       | Yes                |

<center><img src="https://img2022.cnblogs.com/blog/2122768/202206/2122768-20220603161145230-2108152099.png" style="zoom:75%"></center>

支持的S3生命周期转换
<center><img src="https://img2022.cnblogs.com/blog/2122768/202206/2122768-20220611224609418-195850411.png" style="zoom:88%"></center>

- 默认终止策略：按需与 Spot 任何分配策略的实例 -> 最旧启动配置的实例 -> 最旧启动模板的实例 -> 最接近下一个计费小时
- 通过 Cognito 用户池为您的应用程序负载均衡器使用 Cognito 身份验证
- EFS Bursting Throughput mode 吞吐量会随着标准存储类中文件系统大小的增长而扩展；预置吞吐量模式，您可以立即预置文件系统的吞吐量（以 MiB/s 为单位），而与存储的数据量无关
- 如果您为 Lambda 函数创建的 IAM 角色与存储桶位于同一 AWS 账户中，则您无需同时授予 Amazon S3 对 IAM 角色和存储桶策略的权限。需要授予 IAM 角色的权限，然后验证存储桶策略没有明确拒绝对 Lambda 函数角色的访问。如果 IAM 角色和存储桶位于不同的账户中，则您需要授予 Amazon S3 对 IAM 角色和存储桶策略的权限。
- "KMS" - AWS 密钥管理服务 (AWS KMS) 是一项服务，它结合了安全、高度可用的硬件和软件，以提供针对云扩展的密钥管理系统。当您将服务器端加密与 AWS KMS (SSE-KMS) 一起使用时，您可以指定您已创建的客户托管 CMK。SSE-KMS 为您提供审计跟踪，显示您的 CMK 的使用时间和用户。KMS 是一种加密服务，它不是秘密存储。所以这个选项是不正确的。
- "CloudHSM"- AWS CloudHSM 是一个基于云的硬件安全模块 (HSM)，使您能够轻松地在 AWS 云上生成和使用您的加密密钥。借助 CloudHSM，您可以使用经过 FIPS 140-2 3 级验证的 HSM 管理您的加密密钥。CloudHSM 符合标准，使您能够将所有密钥导出到大多数其他市售 HSM，具体取决于您的配置。它是一项完全托管的服务，可为您自动执行耗时的管理任务，例如硬件配置、软件修补、高可用性和备份。CloudHSM 也是一种加密服务，而不是秘密存储。所以这个选项是不正确的。
- AWS Systems Manager Parameter Store（又名 SSM Parameter Store）为配置数据管理和机密管理提供安全的分层存储。您可以将密码、数据库字符串、EC2 实例 ID、Amazon 系统映像 (AMI) ID 和许可证代码等数据存储为参数值。您可以将值存储为纯文本或加密数据。您可以使用创建参数时指定的唯一名称在脚本、命令、SSM 文档以及配置和自动化工作流中引用 Systems Manager 参数。SSM Parameter Store 可以用作秘密存储，但您必须自己轮换秘密，它没有自动执行此操作的功能。所以这个选项是不正确的。
- Amazon Kinesis Data Firehose 是将流数据加载到数据存储和分析工具中的最简单方法。它可以捕获、转换流数据并将其加载到 Amazon S3、Amazon Redshift、Amazon Elasticsearch Service 和 Splunk 中，使用您现在已经在使用的现有商业智能工具和仪表板实现近乎实时的分析。
- Amazon Kinesis Data Analytics 是实时分析流数据的最简单方法。您可以使用内置模板和运算符快速构建 SQL 查询和复杂的 Java 应用程序，以实现通用处理功能，以组织、转换、聚合和分析任何规模的数据。Kinesis Data Analytics 使您能够通过三个简单的步骤轻松快速地构建查询和复杂的流式处理应用程序：设置流式处理数据源、编写查询或流式处理应用程序以及设置处理数据的目的地。
- [Alias 和 CNAME 的比较](https://docs.aws.amazon.com/zh_cn/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html)
- ALB不能分配弹性IP，NLB可以
- 如果您的实例未能通过系统状态检查，您可以使用 CloudWatch 警报操作来自动恢复它。恢复选项适用于超过 90% 的已部署客户 EC2 实例。CloudWatch 恢复选项仅适用于系统检查失败，不适用于实例状态检查失败。此外，如果您终止您的实例，则它无法恢复。不能使用 CloudWatch 事件直接触发 EC2 实例的恢复。仅在配置了 EBS 卷的实例上支持恢复操作，CloudWatch 警报不支持实例存储卷进行自动恢复。
- 恢复的实例与原始实例相同，包括实例 ID、私有 IP 地址、弹性 IP 地址和所有实例元数据，如果您的实例有公有 IPv4 地址，它会在恢复后保留公有 IPv4 地址
- 对于大型对象的长距离传输，Amazon S3 Transfer Acceleration 可以将进出 Amazon S3 的内容传输速度提高 50-500%。拥有广泛用户的 Web 或移动应用程序或托管在远离其 S3 存储桶的应用程序的客户可以通过 Internet 体验长时间且可变的上传和下载速度。S3 传输加速 (S3TA) 减少了 Internet 路由、拥塞和可能影响传输的速度的可变性，并从逻辑上缩短了远程应用程序到 S3 的距离。小于 1GB 的对象或数据集的大小小于 1GB，则考虑使用CloudFront。
- 您可以使用网络地址转换 (NAT) 网关使私有子网中的实例能够连接到 Internet 或其他 AWS 服务，但阻止 Internet 启动与这些服务的连接实例。要创建 NAT 网关，您必须指定 NAT 网关应驻留的公共子网。您还必须在创建 NAT 网关时指定一个弹性 IP 地址以与它关联。与 NAT 网关关联后，弹性 IP 地址无法更改。创建 NAT 网关后，您必须更新与一个或多个私有子网关联的路由表，以将 Internet 绑定流量指向 NAT 网关。这使您的私有子网中的实例能够与 Internet 通信。如果您不再需要 NAT 网关，可以将其删除。删除 NAT 网关会解除其弹性 IP 地址的关联，但不会从您的账户中释放该地址。仅出口互联网网关是支持 IPv6 流量的互联网网关，Internet 网关不能直接与私有子网一起使用。如果公有子网中没有 NAT 实例或 NAT 网关，则无法设置此配置。
- AWS DataSync 是一种在线数据传输服务，可简化、自动化和加速通过 Internet 或 AWS Direct Connect 与 AWS 存储服务之间的大量数据复制。
- FIFO 队列支持每秒最多 3,000 条消息；FIFO 队列的名称必须以 .fifo 后缀结尾
- EFS、EBS、S3不支持 SMB 协议。AWS Storage Gateway和Fsx支持。
- Amazon VPC 控制台向导提供以下四种配置：具有单个公有子网的 VPC、具有公有和私有子网 (NAT) 的 VPC、具有公有和私有子网以及 AWS Site-to-Site VPN 访问的 VPC
- SCP 权限的影响：如果用户或角色的 IAM 权限策略授予对适用 SCP 不允许或明确拒绝的操作的访问权限，则用户或角色无法执行该操作。SCP 会影响附加账户中的所有用户和角色，包括 root 用户。SCP 不影响任何服务相关角色。
- 使用 AWS Systems Manager，您可以按应用程序对资源（如 Amazon EC2 实例、Amazon S3 存储桶或 Amazon RDS 实例）进行分组，查看操作用于监控和故障排除的数据，并对您的资源组采取行动。您不能使用 Systems Manager 来维护资源配置更改的历史记录。
- 创建启动配置时，实例放置租期的默认值为 null，并且实例租期由 VPC 的租期属性控制。如果您将 Launch Configuration Tenancy 设置为默认值并且 VPC Tenancy 设置为 dedicated，则实例具有专用租赁。如果您将 Launch Configuration Tenancy 设置为 dedicated 并且 VPC Tenancy 设置为默认值，则实例再次具有专用租赁。
- Elastic Fabric Adapter (EFA) 是一种网络设备，您可以将其连接到您的 Amazon EC2 实例以加速高性能计算 (HPC) 和机器学习应用程序。它增强了实例间通信的性能，这对于扩展 HPC 和机器学习应用程序至关重要。EFA 设备提供所有 Elastic Network Adapter (ENA) 设备功能以及新的操作系统旁路硬件接口，允许用户空间应用程序直接与硬件提供的可靠传输功能进行通信。
- Elastic Network Adapter (ENA) 设备通过单根 I/O 虚拟化 (SR-IOV) 支持增强网络，以提供高性能网络功能。尽管增强型网络提供了更高的带宽、更高的每秒数据包 (PPS) 性能和持续更低的实例间延迟，但 EFA 仍然更适合给定的用例，因为 EFA 设备提供了 ENA 设备的所有功能，此外硬件支持应用程序直接与 EFA 设备通信，而不涉及使用扩展编程接口的实例内核（OS 旁路通信）。
- 要创建 NAT 网关，您必须指定 NAT 网关应驻留的公共子网。您还必须在创建 NAT 网关时指定一个弹性 IP 地址以与它关联。与 NAT 网关关联后，弹性 IP 地址无法更改。创建 NAT 网关后，您必须更新与一个或多个私有子网关联的路由表，以将 Internet 绑定流量指向 NAT 网关。这使您的私有子网中的实例能够与 Internet 通信。
- NAT 实例可用作堡垒服务器、安全组可以与 NAT 实例关联、NAT实例支持端口转发
- 您只能在启动实例后将实例的租期从专用更改为主机，或从主机更改为专用。
- AWS OpsWorks - AWS OpsWorks 是一种配置管理服务，提供 Chef 和 Puppet 的托管实例。Chef 和 Puppet 是自动化平台，允许您使用代码来自动化服务器的配置。OpsWorks 允许您使用 Chef 和 Puppet 来自动化跨 Amazon EC2 实例或本地计算环境配置、部署和管理服务器的方式。