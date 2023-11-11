### 1. CloudFront vs S3 Cross Region Replication
CloudFront:

- 全球边缘网络
- 文件缓存一个TTL（可能一天）
- **非常适合必须随处可用的静态内容**

S3 Cross Region Replication:
- 必须为要进行复制的每个区域设置
- 文件几乎实时更新
- 只读
- **非常适合在少数地区以低延迟提供的动态内容**

### 2. CloudFront Signed URL Diagram

<center><img src="signedurl.png" style="zoom:55%"></center>

### 3. CloudFront Signed URL vs S3 Pre-Signed URL
**CloudFront Signed URL:**

- 允许访问路径，无论其来源如何
- 帐户范围的密钥对，只有root用户才能管理
- 可以按IP、路径、日期、过期日期进行筛选
- 可以利用缓存功能

**S3 Pre-Signed URL:**

- 以预签名URL的身份发出请求
- 使用签名IAM主体的IAM密钥
- 寿命有限

### 4. Unicast IP vs Anycast IP

- 单播IP：一台服务器拥有一个IP地址
- Anycast IP：所有服务器都拥有相同的IP地址，客户端路由到最近的一个

### 5. AWS Global Accelerator vs CloudFront
他们都使用AWS全球网络及其在世界各地的边缘位置
这两项服务都与AWS Shield集成以提供DDoS保护。
CloudFront

- 提高了可缓存内容（如图像和视频）的性能
- 动态内容（如API加速和动态网站交付）
- 边缘提供内容

Global Accelerator
- 通过TCP或UDP提高各种应用程序的性能
- 将边缘的数据包代理到一个或多个AWS区域中运行的应用程序。
- 非常适合非HTTP用例，如游戏（UDP）、物联网（MQTT）或IP语音
- 适用于需要静态IP地址的HTTP用例
- 适用于需要确定性、快速区域故障切换的HTTP用例

使用SQS:队列模型
使用SNS:发布/子模型
使用Kinesis：实时流媒体模型

### 6. Amazon SQS – Standard Queue

- 最老的产品（超过10年）
- 完全管理的服务，用于**解耦应用程序**
- 属性：
  - 吞吐量不受限制，队列中的消息数量不受限制
  - 邮件的默认保留期：4天，最长14天
  - 低延迟（发布和接收时＜10毫秒）
  - 发送的每条消息限制为256KB
- 可能有重复的邮件（至少一次传递，偶尔）
- 可能会出现无序消息（尽力订购）

### 7. Kinesis
- Kinesis数据流：捕获、处理和存储数据流
- Kinesis Data Firehose：将数据流加载到AWS数据存储中以可靠方式将实时数据串流加载到数据湖、数据库和分析服务中
- Kinesis数据分析：使用SQL或Apache Flink分析数据流
- Kinesis视频流：捕获、处理和存储视频流