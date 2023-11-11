### 1. What's an EBS Volume?

- **它们一次只能挂载到一个实例**（CCP级别）
- 它们绑定到**特定的可用性区域**

### 2. EBS – 终止时删除属性

- 控制EC2实例终止时的EBS行为
  - 默认情况下，根EBS卷被删除（属性已启用）
  - 默认情况下，不会删除任何其他连接的EBS卷（禁用属性）
  
- 这可以通过AWS控制台/AWS CLI进行控制
- **用例：实例终止时保留根卷**

### 3. EBS Snapshots

- 在某个时间点备份EBS卷（快照）
- 不需要分离卷来进行快照，但建议
- 可以跨AZ或Region复制快照

### 4. AMI (Amazon Machine Image)

- AMI是EC2实例的**自定义**
  - 您添加自己的软件、配置、操作系统、监控…
  - 更快的启动/配置时间，因为您的所有软件都是预打包的
- AMI是为**特定区域**构建的（并且可以跨区域复制）
- 您可以从以下位置启动EC2实例：
  - **公共AMI**：提供AWS
  - **你自己的AMI**：你自己制作和维护它们
  - **AWS Marketplace AMI**：其他人制造（并可能销售）的AMI

### 5. AMI Process (from an EC2 instance)

- 启动EC2实例并进行自定义
- 停止实例（为了数据完整性）
- 构建AMI–这也将**创建EBS快照**
- 从其他AMI启动实例

### 6. EBS Volume Types

- EBS卷有6种类型
  - <font color=blue>gp2/gp3（SSD）</font>：通用SSD卷，可平衡各种工作负载的性价比
  - <font color=blue>io1/io2（SSD）</font>：用于任务关键型低延迟或高吞吐量工作负载的最高性能SSD卷
  - <font color=blue>st1（HDD）</font>：低成本HDD卷，专为频繁访问、吞吐量密集型工作负载而设计
  - <font color=blue>sc1（HDD）</font>：为访问频率较低的工作负载设计的成本最低的HDD卷
- EBS卷的特征是大小|吞吐量| IOPS（每秒I/O操作数）
- 如果有疑问，请务必查阅AWS文档——这很好！
- **只有gp2/gp3和io1/io2可以用作启动卷**

### 7. EBS –Volume Types Summary
[<font color=blue>Amazon EBS卷类型</font>](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-volume-types.html)

### 8. EBS Multi-Attach – io1/io2 family

- 将同一EBS卷附加到同一AZ中的多个EC2实例
- 每个实例都具有对卷的完全读写权限
- 用例：
  - 在集群Linux应用程序中实现**更高的应用程序可用性**（例如：Teradata）
  - 应用程序必须管理并发写入操作
- 必须使用支持集群的文件系统（不是XFS、EX4等）

### 9. EBS Encryption

- 创建加密EBS卷时，您会得到以下信息：
  - 静止数据在卷内加密
  - 在实例和卷之间移动的所有数据都是加密的
  - 所有快照都已加密
  - 从快照创建的所有卷
- 加密和解密是透明处理的（您无需做任何事情）
- 加密对延迟的影响最小
- EBS加密利用KMS（AES-256）的密钥
- 复制未加密的快照允许加密
- 加密卷的快照已加密

### 10. 加密未加密的EBS卷

- 创建卷的EBS快照
- 加密EBS快照（使用副本）
- 从快照创建新ebs卷（该卷也将被加密）
- 现在您可以将加密卷附加到原始实例

### 11. EFS – Elastic File System

- 可安装在许多EC2上的托管NFS（网络文件系统）
- EFS与多AZ中的EC2实例一起工作
- 高可用性、可扩展性、价格昂贵（3倍gp2）、按次付费
- 使用案例：内容管理、网络服务、数据共享、Wordpress
- 使用NFSv4.1协议
- 使用安全组控制对EFS的访问
- **与基于Linux的AMI（非Windows）兼容**
- 使用KMS进行静态加密
- POSIX（可移植操作系统接口可移植操作系统接口）具有标准文件API的文件系统（~Linux）
- 文件系统自动扩展，按次付费，无需容量规划！

### 12. EFS – Performance & Storage Classes

- **EFS Scale**
  - 1000个并发NFS客户端，10 GB+/s吞吐量
  - 自动扩展到Petabyte规模的网络文件系统
- **Performance mode (set at EFS creation time)**
  - 通用（默认）：延迟敏感用例（web服务器、CMS等）
  - 最大I/O–更高的延迟、吞吐量、高度并行（大数据、媒体处理）
- **Throughput mode**
  - 突发（1 TB=50MiB/s+高达100MiB/s的突发）
  - Provisioned：无论存储大小如何，都可以设置吞吐量，例如：1 TB存储为1 GiB/s
- **Storage Tiers (lifecycle management feature – move file after N days)**
  - 标准：用于频繁访问的文件
  - <font color=blue>不频繁访问（EFS-IA）</font>：检索文件的成本更低，存储成本更低

### 13. EBS vs EFS – Elastic Block Storage

- EBS volumes…
  - 一次只能附加到一个实例
  - 锁定在可用区（AZ）级别
  - gp2:IO在磁盘大小增加时增加
  - io1：可以独立增加IO
- 跨AZ迁移EBS卷
  - 拍摄快照
  - 将快照恢复到另一个AZ
  - EBS备份使用IO，您不应该在应用程序处理大量流量时运行它们
- 如果EC2实例被终止，实例的根EBS卷默认情况下会被终止（您可以禁用它）

### 14. EBS vs EFS – Elastic File System

- 在AZ上安装100个实例
- EFS共享网站文件（WordPress）
- 仅适用于Linux实例（POSIX）
- EFS的价位高于EBS
- 可以利用EFS-IA节省成本