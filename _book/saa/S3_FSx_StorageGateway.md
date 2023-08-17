### 1. Amazon S3 Overview - Buckets

- Amazon S3 allows people to store objects (files) in "buckets" (directories)
- Buckets must have a **globally unique name**
- Buckets are defined at the region level 
- Naming convention
  - No uppercase
  - No underscore
  - 3-63 characters long
  - Not an IP
  - Must start with lowercase letter or number

### 2. Amazon S3 Overview – Objects

- Object values are the content of the body:
  - Max Object Size is 5TB (5000GB)
  - If uploading more than 5GB, must use "multi-part upload"
- Metadata (list of text key / value pairs – system or user metadata)
- Tags (Unicode key / value pair – up to 10) – useful for security / lifecycle
- Version ID (if versioning is enabled)

### 3. S3 Encryption for Objects
SSE-S3: encrypts S3 objects using keys handled & managed by AWS

- Object is encrypted server side
- AES-256 encryption type
- Must set header: **"x-amz-server-side-encryption": "AES256"**

SSE-KMS: leverage AWS Key Management Service to manage encryption keys
- SSE-KMS: encryption using keys handled & managed by KMS
- KMS Advantages: user control + audit trail
- Object is encrypted server side
- Must set header: **"x-amz-server-side-encryption": "aws:kms"**

SSE-C: when you want to manage your own encryption keys
- server-side encryption using data keys fully managed by the customer outside of AWS
- Amazon S3 does not store the encryption key you provide
- **HTTPS must be used**
- Encryption key must provided in HTTP headers, for every HTTP request made

Client Side Encryption
- Client library such as the Amazon S3 Encryption Client 
- Clients must encrypt data themselves before sending to S3 
- Clients must decrypt data themselves when retrieving from S3 
- Customer fully manages the keys and encryption cycle

### 4. S3 Security
**User based**

- IAM policies - which API calls should be allowed for a specific user from IAM console

**Resource Based**
- Bucket Policies - bucket wide rules from the S3 console - allows cross account
- Object Access Control List (ACL) – finer grain
- Bucket Access Control List (ACL) – less common

**Note**: an IAM principal can access an S3 object if 
- the user IAM permissions allow it **OR** the resource policy ALLOWS it
- **AND** there's no explicit DENY

### 5. CORS (Cross-Origin Resource Sharing)
using **CORS Headers (ex: Access-Control-Allow-Origin)**

### 6. AWS EC2 Instance Metadata

- AWS EC2 Instance Metadata is powerful but one of the least known features to developers
- It allows AWS EC2 instances to "learn about themselves" **without using an IAM Role for that purpose**.
- The URL is [<font color=blue>http://169.254.169.254/latest/meta-data</font>](http://169.254.169.254/latest/meta-data)
- You can retrieve the IAM Role name from the metadata, but you CANNOT retrieve the IAM Policy

### 7. Amazon FSx

- Launch 3rd party high-performance file systems on AWS
- Fully managed service

### 8. Amazon FSx for Windows (File Server)

- EFS is a shared POSIX system for Linux systems.
- **FSx for Windows** is a fully managed **Windows** file system share drive
- Supports SMB protocol & Windows NTFS
- Microsoft Active Directory integration, ACLs, user quotas
- Built on SSD, scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Can be accessed from your on-premise infrastructure
- Can be configured to be Multi-AZ (high availability)
- Data is backed-up daily to S3

### 9. Amazon FSx for Lustre

- Lustre is a type of parallel distributed file system, for large-scale computing
- The name Lustre is derived from "Linux" and "cluster"
- Machine Learning, **High Performance Computing (HPC)**
- Video Processing, Financial Modeling, Electronic Design Automation
- Scales up to 100s GB/s, millions of IOPS, sub-ms latencies
- **Seamless integration with S3**
  - Can "read S3" as a file system (through FSx)
  - Can write the output of the computations back to S3 (through FSx)
- Can be used from on-premise servers

### 10. AWS Storage Gateway

- Bridge between on-premises data and cloud data in S3
- Use cases: disaster recovery, backup & restore, tiered storage

### 11. File Gateway

- Configured S3 buckets are accessible using the NFS and SMB protocol
- Supports S3 standard, S3 IA, S3 One Zone IA
- Bucket access using IAM roles for each File Gateway
- Most recently used data is cached in the file gateway
- Can be mounted on many servers
- **Integrated with Active Directory (AD)** for user authentication

### 12. Volume Gateway

- Block storage using iSCSI protocol backed by S3
- Backed by EBS snapshots which can help restore on-premises volumes!
- **Cached volumes**: low latency access to most recent data
- **Stored volumes**: entire dataset is on premise, scheduled backups to S3

### 13. Tape Gateway

- Some companies have backup processes using physical tapes (!)
- With Tape Gateway, companies use the same processes but, in the cloud
- Virtual Tape Library (VTL) backed by Amazon S3 and Glacier
- Back up data using existing tape-based processes (and iSCSI interface)
- Works with leading backup software vendors

### 14. Storage Comparison
- **S3**: Object Storage
- **Glacier**: Object Archival
- **EFS**: Network File System for Linux instances, POSIX filesystem
- **FSx for Windows**: Network File System for Windows servers
- **FSx for Lustre**: High Performance Computing Linux file system
- **EBS volumes**: Network storage for one EC2 instance at a time
- **Instance Storage**: Physical storage for your EC2 instance (high IOPS)
- **Storage Gateway**: File Gateway, Volume Gateway (cache & stored), Tape Gateway
- **Snowball / Snowmobile**: to move large amount of data to the cloud, physically
- **Database**: for specific workloads, usually with indexing and querying