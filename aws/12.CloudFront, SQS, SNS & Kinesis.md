###CloudFront vs S3 Cross Region Replication
CloudFront:
- Global Edge network
- Files are cached for a TTL (maybe a day)
- **Great for static content that must be available everywhere**

S3 Cross Region Replication:
- Must be setup for each region you want replication to happen
- Files are updated in near real-time
- Read only
- **Great for dynamic content that needs to be available at low-latency in few regions**

###CloudFront Signed URL Diagram
<center><img src="https://img2022.cnblogs.com/blog/2122768/202204/2122768-20220423110506487-1603730261.png" style="zoom:55%"></center>

###CloudFront Signed URL vs S3 Pre-Signed URL
**CloudFront Signed URL:**
- Allow access to a path, no matter the origin
- Account wide key-pair, only the root can manage it
- Can filter by IP, path, date, expiration
- Can leverage caching features

**S3 Pre-Signed URL:**
- Issue a request as the person who pre-signed the URL
- Uses the IAM key of the signing IAM principal
- Limited lifetime

###Unicast IP vs Anycast IP
- Unicast IP: one server holds one IP address
- Anycast IP: all servers hold the same IP address and the client is routed to the nearest one

###AWS Global Accelerator vs CloudFront
They both use the AWS global network and its edge locations around the world
Both services integrate with AWS Shield for DDoS protection.
CloudFront
- Improves performance for both cacheable content (such as images and videos)
- Dynamic content (such as API acceleration and dynamic site delivery)
- Content is served at the edge

Global Accelerator
- Improves performance for a wide range of applications over TCP or UDP
- Proxying packets at the edge to applications running in one or more AWS Regions.
- Good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
- Good for HTTP use cases that require static IP addresses
- Good for HTTP use cases that required deterministic, fast regional failover

using SQS: queue model
using SNS: pub/sub model
using Kinesis: real-time streaming model

###Amazon SQS – Standard Queue
- Oldest offering (over 10 years old)
- Fully managed service, used to **decouple applications**
- Attributes:
  - Unlimited throughput, unlimited number of messages in queue
  - Default retention of messages: 4 days, maximum of 14 days
  - Low latency (<10 ms on publish and receive)
  - Limitation of 256KB per message sent
- Can have duplicate messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)

###Kinesis
- Kinesis Data Streams: capture, process, and store data streams
- Kinesis Data Firehose: load data streams into AWS data stores 以可靠方式将实时数据串流加载到数据湖、数据库和分析服务中
- Kinesis Data Analytics: analyze data streams with SQL or Apache Flink
- Kinesis Video Streams: capture, process, and store video streams