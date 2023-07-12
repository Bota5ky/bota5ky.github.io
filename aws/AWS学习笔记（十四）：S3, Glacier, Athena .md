###S3 MFA-Delete
- MFA (multi factor authentication) forces user to generate a code on a device (usually a mobile phone or hardware) before doing important operations on S3
- To use MFA-Delete, enable Versioning on the S3 bucket
- You will need MFA to
  - permanently delete an object version
  - suspend versioning on the bucket
- You won’t need MFA for
  - enabling versioning
  - listing deleted versions
- **Only the bucket owner (root account) can enable/disable MFA-Delete**
- MFA-Delete currently can only be enabled using the CLI
- ***Note:** Bucket Policies are evaluated before "default encryption"

###S3 Replication (CRR & SRR)
- **Must enable versioning** in source and destination
- Cross Region Replication (CRR)
- Same Region Replication (SRR)
- Buckets can be in different accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3
- CRR - Use cases: compliance, lower latency access, replication across accounts
- SRR – Use cases: log aggregation, live replication between production and test accounts

###S3 Replication – Notes
- After activating, only new objects are replicated (not retroactive)
- For DELETE operations:
  - Can replicate delete markers from source to target (optional setting)
  - Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no "chaining" of replication
  - If bucket 1 has replication into bucket 2, which has replication into bucket 3
  - Then objects created in bucket 1 are not replicated to bucket 3

###S3 Pre-Signed URLs
- Can generate pre-signed URLs using SDK or CLI
  - For downloads (easy, can use the CLI)
  - For uploads (harder, must use the SDK)
- Valid for a default of 3600 seconds, can change timeout with --expires-in [TIME_BY_SECONDS] argument
- Users given a pre-signed URL inherit the permissions of the person who generated the URL for GET / PUT

###Amazon S3存储类别
[S3 Storage Classes Comparison](https://aws.amazon.com/cn/s3/storage-classes/)
<img src="https://img2022.cnblogs.com/blog/2122768/202204/2122768-20220420204807511-924591363.png" style="zoom:60%">

###S3 – Moving between storage classes
<center><img src="https://img2022.cnblogs.com/blog/2122768/202204/2122768-20220420205726702-2012779652.png" style="zoom:65%"></center>

###S3 Lifecycle Rules
- **Transition actions**: It defines when objects are transitioned to another storage class.
  - Move objects to Standard IA class 60 days after creation
  - Move to Glacier for archiving after 6 months
- **Expiration actions**: configure objects to expire (delete) after some time
  - Access log files can be set to delete after a 365 days
  - **Can be used to delete old versions of files (if versioning is enabled)**
  - Can be used to delete incomplete multi-part uploads
- Rules can be created for a certain prefix (ex - s3://mybucket/mp3/*)
- Rules can be created for certain objects tags (ex - Department: Finance)

###S3 Analytics – Storage Class Analysis
- You can setup S3 Analytics to help determine when to transition objects from Standard to Standard_IA
- Does not work for ONEZONE_IA or GLACIER
- Report is updated daily
- Takes about 24h to 48h hours to first start
- Good first step to put together Lifecycle Rules (or improve them)!

###S3 – Baseline Performance
- Amazon S3 automatically scales to high request rates, latency 100-200 ms
- Your application can achieve at least **3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix in a bucket**

###S3 Performance
- **Multi-Part upload:**
  - recommended for files > 100MB, must use for files > 5GB
  - Can help parallelize uploads (speed up transfers)
- **S3 Transfer Acceleration**
  - Increase transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region
  - Compatible with multi-part upload

###Athena
one time SQL queries, serverless queries on S3, log analytics