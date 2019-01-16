# AWS Solution Architect February 2018 Associate Level Certification Exam Notes

## IAM

## S3
### S3 Usage Patterns (When to use S3)
1. Store and distribute static web content and media
2. Host entire static web sites
3. Data store for computation and large scale analytics
4. Use as a highly durable, scalable, and secure solution for backup and archiving of critical data

### When NOT to use S3:
1. As a file system
  - Use AWS EFS instead
2. Strucured data with queries
  - Use RDS, DynamoDB or Amazon CloudSearch
3. Rapidly/Frequently Changing Data
  - Use RDS, DynamoDB, EFS or Amazon CloudSearch
4. Archival
  - Use Amazon Glacier
5. Dynamic Website Hosting
  - Use EC2 or EFS
  
### S3 Cost Model
Amazon S3 Standard has three pricing components: 
* storage (per GB per month) 
* data transfer in or out (per GB per month)
* requests (per thousand requests per month)

### S3 Data Consistency Model
**Read-after-write** for HTTP GET/LIST after object is written to S3 bucket successfully. However, if HTTP GET/HEAD request is made before object is written to S3 bucket, then S3 ensures **eventually consistent data moedl for read-after-write**.

**Eventual consistency** also applies to overwrite PUTs and DELETEs in all regions. This is due to S3's high availability (replication) takes some time to propogate the PUT/DELETE overwrite across all AZs.

### Interfaces
1. **AWS Console:**
Max file size that can be uploaded to S3 from AWS console is **78 GB**
2. **AWS CLI** commands/scripts
3. **AWS SDKs and REST API calls**
4. **AWS Services:**
  4.1. AWS Direct Connect
  4.2. AWS Storage Gateway
  4.3. Amazon Kenesis Data Firehose
  4.4. Amazon Kenesis Video Streams
  4.5. Amazon Kenesis Data Streams
  4.6. Amazon S3 Transfer Acceleration
  4.7. AWS Snowball
  4.8. AWS Snowball Edge
  4.9. AWS Snowmobile
  4.10. Third Party Connectors

### Data Access
1. REST APIs to access buckets and objects
  1.1 **Path-style URL:** E.g `http://s3-eu-west-1.amazonaws.com/mybucket/image01.jpg`
  > **NOTE:** For US East (N. Viriginia) region, the URL will be different. E.g. `http://s3.amazonaws.com/mybucket/image01.jpg`
  1.2 **Virtual-hosted-style URL:** These are more user friendly as they starts with bucket name E.g. `http://mybucket.s3.amazonaws.com/image01.jpg`
  * Using custom domain names to access S3 bucket: 
      **HTTP-based URL:** Match bucket name to DNS registry name. E.g. `http://www.example.com/device.html` 
      **HTTPS-based URL:** DON'T use *periods* in bucket name as it will NOT work with SSL due to SSL certificate exceptions. To avoid this issue- Write your own SSL certificate verification rule **OR** use HTTP-based virtual-hosted-URLs **OR** use path-style URLs **OR** use  `Amazon CloudFront` in conjunction with S3.

> S3 uses DNS for routing requests.

### Handling Routing errors:
> If a request to S3 is incorrectly routed to incorrect AWS region, S3 sends temporary redirect. **Ensure to implement retry logic in you requester application for redirect response codes**.

> If a request to S3 is mal-formed, S3 sends **permanent redirect** and responds with 4XX bad request error code. Fix the request to resolve this issue.

## EC2

## AWS services that are specific to a region
The below AWS services are specific to a AWS region. E.g. If you plan to launch AWS EC2 instances in multiple regions, you'll need to create a security group in each region.

1. Security Groups
2. IAM Keys
3. 
