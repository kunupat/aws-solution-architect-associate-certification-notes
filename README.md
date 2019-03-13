# AWS Solutions Architect February 2018 Associate Level Certification Exam Notes

Refer my [GitHub Pages site](https://kunupat.github.io/2019/01/16/AWS-Solutions-Architect-Associate-Certification.html) for a listing of material and useful links to pass AWS Solutions Architect February 2018 Associate Level Certification exam.

## Contents
* [Identity and Access Management (IAM)](#identity-and-access-management)
* [Simple Storage Service (S3)](#simple-storage-service)
* [Storage Gateway](#storage-gateway)
* [Snowball](#snowball)
* [Elastic Compute Cloud (EC2)](#elastic-compute-cloud)
* [Autoscaling](#autoscaling)
* [Elastic Block Storage (EBS)](#elastic-block-storage)
* [Elastic Load Balancer (ELB)](#elastic-load-balancer)
* [Elastic File System (EFS)](#elastic-file-service)
* [Lambda Functions](#lambda-functions)
* [Cloudwatch](#cloudwatch)
* [Virtual Private Cloud (VPC) And Other Services](#virtual-private-cloud-and-other-services)
* [Route 53](#route-53)
* [Databases](#databases)
* [Simple Queue Service (SQS)](#simple-queue-service)
* [Simple Workflow Service (SWF)](#simple-workflow-service)
* [Simple Notification Service (SNS)](#simple-notification-service)
* [Elastic Transcoder](#elastic-transcoder)
* [API Gateway](#api-gateway)
* [Kinesis](#kinesis)
* [CloudFormation](#cloudformation)
* [Elastic Container Service(ECS)](#elastic-container-service)
* [Other Notes](#other-notes)
* [Exam structure in January 2019](#exam-structure-in-january-2019)

## Identity and Access Management
### IAM Policies

> Control what this user can do in AWS.

IAM policies can be assigned to IAM users, groups and roles.

Sample IAM policy for allowing PUT object action on Amazon S3 bucket:

```
{
  "Version":"2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::mybucket/*"
    }
  ]
}
```
> **Note:** IAM policy has three main parts: Action, Effect and Resource. An IAM policy DOESN'T have principal.

### IAM Roles
- Global service
- Use IAM Roles instead of using AWS access and private key to access AWS services programmatically (using CLI, API, etc.)
  - E.g. Attach an IAM role to EC2 instance that allows admin access to S3. Then, when you SSH into this EC2 instance, you can run AWS CLI commands like `aws s3 ls` from this EC2 instance without needing to configure AWS CLI with the AWS access key and private key(
- IAM Role can be attached to a running EC2 instance too now

## Simple Storage Service
- Object based storage (0TB to 5TB file sizes)
- Unlimited Storage
- All objects(files) are stored in buckets and bucket namespaces have to be unique universally.
- S3 Bucket Format: `https://s3-<aws_region>.amazonaws.com/<bucket-name>`

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

### S3 Storage Classes
1. **S3 Standard:** 99.99% availability, 99.999999999% durability, can sustain loss of 2 AZs concurrently
2. **S3 Infrequently Accessed (S3-IA):** For data less frequently accessed, but needs faster retrieval when required. Charged for retrieval fee.
3. **S3 Infrequently Accessed One Zone (S3 One zone-IA):** S3-IA with no redundancy as it spans in only one AZ.
4. **Glacier:** For data archival. Typical retrieval time will be 3-5 hours.
5. ~~**Reduced Redundancy Storage (S3 RRS)~~(Deprecated. Use S3 One zone- IA instead):** Reduced Redundancy Storage (RRS) is an Amazon S3 storage option that enables customers to store noncritical, reproducible data at lower levels of redundancy than Amazon S3’s standard storage. Designed to provide 99.99% durability and 99.99% availability of objects over a given year. Designed to sustain the loss of data in a single facility.

### S3 Data Consistency Model
**Read-after-write** for HTTP `GET`/`LIST` operation after object is written(`PUT`) to S3 bucket successfully. However, if HTTP `GET`/`HEAD` request is made before object is written to S3 bucket, then S3 ensures **eventually consistent data model for read-after-write**.

**Eventual consistency** also applies to **overwrite** `PUT`s and `DELETE`s in all regions. This is due to S3's high availability (replication) takes some time to propogate the PUT/DELETE overwrite across all AZs.

### Interfaces
1. **AWS Console:**
Max file size that can be uploaded to S3 from AWS console is **78 GB**
2. **AWS CLI** commands/scripts
3. **AWS SDKs and REST API calls**
4. **AWS Services:**
  - AWS Direct Connect
  - AWS Storage Gateway
  - Amazon Kenesis Data Firehose
  - Amazon Kenesis Video Streams
  - Amazon Kenesis Data Streams
  - Amazon S3 Transfer Acceleration
  - AWS Snowball
  - AWS Snowball Edge
  - AWS Snowmobile
  - Third Party Connectors

### Data Access: REST APIs to access buckets and objects
  - **Path-style URL:** E.g `http://s3-eu-west-1.amazonaws.com/mybucket/image01.jpg`
  > **NOTE:** For US East (N. Viriginia) region, the URL will be different. E.g. `http://s3.amazonaws.com/mybucket/image01.jpg`
  
  - **Virtual-hosted-style URL:** These are more user friendly as they starts with bucket name E.g. `http://mybucket.s3.amazonaws.com/image01.jpg`
  
  **Using custom domain names to access S3 bucket:**
  - **HTTP-based URL:** Match bucket name to DNS registry name. E.g. `http://www.example.com/device.html` 
  - **HTTPS-based URL:** DON'T use *periods* in bucket name as it will NOT work with SSL due to SSL certificate exceptions. To avoid this issue- Write your own SSL certificate verification rule **OR** use HTTP-based virtual-hosted-URLs **OR** use path-style URLs **OR** use  `Amazon CloudFront` in conjunction with S3.

> S3 uses DNS for routing requests.


### Handling Routing errors:
> If a request to S3 is incorrectly routed to incorrect AWS region, S3 sends temporary redirect. **Ensure to implement retry logic in you requester application for redirect response codes**.

> If a request to S3 is mal-formed, S3 sends **permanent redirect** and responds with 4XX bad request error code. Fix the request to resolve this issue.

### Operations on Objects
1. `PUT`
* Single Part- Use for objects of size <= 5GB. **Recommended for objects less than 100 MBs.**
* Multi part- Use for objects of size > 5GB and <= 5TB. **Recommended for objects greater than 100 MBs.** Ensure aborting or completing incomplete mult-part PUTs. There is an option in Lifecycle Management section in AWS console to automatically do this.

2. `COPY`
    Copy objects within S3, rename objects by creating copy and deleting the original, update metadata of object or move objects across S3 locations.

3. `GET`
    Retrieve full object or in multi-part by using **Range GET*

4. `DELETE`
    Delete single or multiple objects with one DELETE. If versioning is disabled, DELETE will permanently deteles the object. If versioning is enabled, S3 can delete object version permanently or insert delete marker. If DETELE request only contains the key name, S3 will insert delete marker and this becomes current version of the object. If you try to GET a object that has delete marker, S3 will respond with 404 NOT FOUND error.

To recover the object, remove delete marker from current version of the object. Delete a specific version of the object by specifying object key and version ID.

To delete the object completly, you MUST detele each individual version.

### Pre-signed URLs
These are used to provide access(PUT/GET) to users/applications who do not have AWS credentials and still not exposing the S3 buckets publicly. These URLs can be programmatically generated using Java/.Net AWS SDK or AWS CLI.

**A pre-signed URL has:** security credentials, bucket name, object key, HTTP method and expiration date-time.

### Cross Origin Resource Sharing (CORS)
CORS can be configured using a XML config file that can contain 100 CORS rules.Use AWS SDK to apply CORS configuration to S3 bucket. CORS configration is used to allow access to S3 objects from an application hosted in different domain.

### S3 Bucket Access
#### Bucket Policies

> Control who can access this bucket.
- Use to make entire bucket public
- Bucket Policies are similar to IAM policy but are applied to AWS resources (S3 in this case). Hence it will also have *principal* defined in the policy as opposed to IAM policies. 
- Bucket policies can also have *conditions*. Condition values can be date, time, ARN of requester, IP of requester, user name, user id and user agent. S3 policy also support conditions using `object tags`.
- Can be used to put size limit policies (upto 20KBs) on S3 buckets/objects.
- Sample bucket policy:

```
{
  "Version":"2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Principal": "aws:arn:iam::123456789012:user/john",
      "Resource": "arn:aws:s3:::mybucket/*"
    }
  ]
}
```
- Bucket policies can be applied to S3 buckets and objects. They CANNOT be used to control access to S3's Management Functions.
- Bucket policy simulator: https://policysim.aws.amazon.com
- Policy Elements:
    * **NotPrincipal:**
        * Denies all except the principal defined in NotPrincipal
        * AWS recommends to not to use it in policy with `Effect=Allow`
        * The order in which AWS evaluates principals makes a difference with `Effect=Deny`
        * Always also provide the AWS account number along with the user ARN which you want to exempt from `deny` policy. If only user ARN is provided in NotPrincipal, then AWS will restrict access to all the users in all those AWS accounts which coontain a user with the given user ARN.
        
    * **NotAction:**
        * `Allow` or `Deny` all actions except one listed with `NotAction`
        * If used with `Deny`, it will not explicitly allow access to all the other actions not listed with `NotAction`
        * If used with `Allow`, it will explicitly allow access to all actions except the one listed with `NotAction`
    * **NotResouce:**
        * `Allow` or `Deny` all resources except one listed with `NotResource`

- Cross-Account Access:
  * **Problem:** Let's say,Person `X` is owner of Bucket `XBucket` created in `Account A`. Person `Y`(from AWS `Account B`) has been allowed access to `PUT` objects in `XBucket` via a bucket policy. When `Y` PUTs an object in `XBucket`, `Y` becomes owner of the object. When `X` tries to `GET` this object, he will be denied access to it even if `X` is the owner of the `XBucket`.
  * **Solution:** Add a `condition` in bucket policy to `allow` full access to bucket owner using `bucket-owner-full-control` when Account B uploads object to `XBucket`. With this, if `PUT` does not grant `bucket-owner-full-control`, the upload will fail.

- Multiple Policy Evaluation: AWS evaluates all policies applied as **OR**. You can define all policies in one policy or in multiple policies. Policies can be attached to a group or to IAM user.

- Access Control Lists: 
  - You may not need IAM policies if ACL are sufficient to control access to buckets.
  - The bucket policy applies only to objects that are owned by the bucket owner. If your bucket contains objects that aren't owned by the bucket owner, public READ permission on those objects should be granted using the object access control list (ACL).
  
- Predefined Groups:
  - Authenticated Users Group: Represents all AWS accounts **worldwide** and not only authenticated users from your AWS account. **USE WITH CAUTION**
  - All Users Group: Open access to all. Requests can be made by authenticated users or anonymous. **USE WITH CAUTION**
  - Log Delivery Group: Use WRITE permission to this group to allow writing server access logs.

> **NOTE:** Use VPC endpoint for secured connection between EC2 instances and S3. Using VPC endpoint, the traffic from EC2 to S3 bucket will not be directed via internet, making it more secured. You can control access to S3 via VPC endpoint by applying VPC endpoint policies OR using bucket policies. VPC endpoint policy is a `resource policy` which means it needs `principal` to be specified.

### Encryption
#### Data At Rest
##### Server Side Encryption
- **AWS provided keys (SSE-S3)**
  - Every object is encrypted with a unique key using `AES-256` encryption standard. Each unique key is encrypted with a regularly rotating master key
  - Enable `Default Encryption` on bucket to enable encryption or use a bucket policy to enable encryption on all the objects stored in a bucket. The `x-amz-server-side-encryption` can be used in bucket policy to `deny` upload to the bucket if the request does not contain `x-amz-server-side-encryption: AES256` request header. In case of using REST API POST method, `x-amz-server-side-encryption: AES256` should be passed as form field and not in request header.
  - We can't enforce SSE-S3 with presigned URLs
  
- **AWS KMS managed keys (SSE-KMS)**
  - Uses a per object key which in turn is encrypted using a rotating customer master key (CMK). Master keys stored in AWS KMS, known as customer master keys (CMKs).
  - This scheme is called [Envelope Encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping)
  - When you use a SSE-KMS for the first time, it will create a CMK automatically. You can create your own CMK as well. Creating a CMK yourself can provide some flexibility to: create, rotate, disable, define access control and audit.
  - Access to KMS keys can be controlled by AWS IAM
  - **NOTE:** There is a restriction on number of requests per second based on the operations. Check the [limits here](http://docs.aws.amazon.com/kms/latest/developerguide/limits.html)
  
- **Customer provided keys (SSE-C)**
  - Amazon S3 does not store your encryption key. You have to manage your encryption keys

##### Default Encryption
It can be either AES-256 or AWS-KMS or None. Any new object will be encrypted with the chosen default encryption.

##### Client Side Encryption
- Encrypt data before uploading to S3. There are two options available:
  - Use AWS KMS-Managed customer master key
  - Use client-side master key
  
#### Data In Transit
- Using SSL/TLS

### Security
- By default, all S3 buckets are PRIVATE
- Available security options can be through-
  - Bucket Policies
  - Access Control Lists (ACL)
- [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
  - Monitors and records S3 configurations. Notifies when config is not compliant to internal guidelines
  - Auditing and SNS notifications
  - Built-in rules:
    - s3-bucket-logging-enabled
    - s3-bucket-public-read-prohibited
    - s3-bucket-public-write-prohibited
    - s3-bucket-ssl-requests-only
    - s3-bucket-versioning-enabled
  - Custom rules  
- [API Logging with AWS Cloudtrail](https://docs.aws.amazon.com/AmazonS3/latest/dev/cloudtrail-logging.html)
- AWS Inventory to review bucket permissions
- **AWS Trusted Advisor** to inspect AWS bucket permissions, bucket logging, bucket versioning. Free if business or enterprise support is enabled on your AWS account.
- [Amazon Macie](https://aws.amazon.com/macie/): Machine learning based service to determine PII, PHI, PCI, etc. data in AWS.

### Bucket Options
1. **Versioning:** Recycle-bean like feature
  - Delete the *Delete Marker* to restore the deleted versioned object
  - **Suspend Versioning:** 
    - `PUT`: If a new version of an object is uploaded, then if that object had any previous versions, a new version of the object will be uploaded with `Null` version ID with old versions intact. If, the object had no previous versions, a new version of the object will be uploaded with `Null` version ID.
    - `DELETE`: If an object had multiple existing versions, `DELETE` operation will create a Detele Marker. When you delete the Detele Marker, it will delete the marker but all previous versions of the object will be retained. If an object had no existing versions, `DELETE` operation will create a Detele Marker. When you delete the Delete Marker, it will delete the marker and the object's version too. 
    
2. **Server Access Logging:**
- Server access logs contain the following:
  - Requester
  - Bucket Name
  - Request Time
  - Request Action
  - Response Status
  - Error Code
- Logs are written to another S3 Bucket or the same bucket on which the Server Access Logging is enabled. Ensure that the destination bucket has right permissions to allow writing logs to it.

3. **Object-level Logging:**
- Using CloudTrail Logging
- Know *Who, When and What* about the requests at the objects and buckets level
> **Recommendation:** If you enable object access logging on all the S3 buckets in your account then choose the destination bucket to use for logging in another AWS account.

4. **Static Website Hosting:** [More Details Here](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)
- No additional service is required to host static website (client-side scripting is supported but NOT server side scripting)
- Format 1: `<bucket-name>.s3-website-<AWS-Region>.amazonaws.com`
- Format 2: `<bucket-name>.s3-website.<AWS-Region>.amazonaws.com`

5. **Default Encryption**

### Managing Storage
- Lifecycle Configuration Rules can be used for Automatic Transition to S3 storage tier.
- S3 Standard to others:
  - objects < 128 KB cannot be transitioned
  - objects must be stored for more than 30 days
- S3 Stardard IA to S3 One-zone IA or Glacier:
  - objects must be stored for more than 30 days
- S3 One-zone IA to Glacier
- S3 Glacier
  - This is one way transition
- S3 Inventory: S3 object metadata can be expoerted as CSV or ORC file. Inventory is cheaper than REST APIs for S3 to generate report of S3 objects.
- Two actions that can be used in a lifecycle policy:
  - Expiration
  - Transition
- Tags
- Events
- Versioning
- Requester Pays
- Server Access Logging
- Default Encryption
- Object Level Logging
- Static Websites
- Transfer Acceleration
- Cross Region Replication(CRR): Can be needed for-
  - Compliance
  - Reduce Latency
  - Operational
  - Data Protection
  - CRR Requirements:
    - Versioning enabled on source and destination buckets
    - Buckets must be in different regions
    - IAM role is configured for CRR
  - CRR does not replicate:
    - bucket lifecycle configuration policies
    - objects in source bucket before the replication config was added
    - objects in source bucket for which the bucket owner does not have permissions for
    - objects in source bucket that are already replicated 
    - SEC-C encrypted objects
  - Ownership Overwrite: Owner of the destination bucket becomes the owner of object after it is replicated
  - Use **Storage Class Analysis** to analyze storage access patterns to use the right storage classes
  - Use **Amazon QuickSight** for storage class analysis
  - Use **Cloudwatch Metrics** for S3:
    - Storage Metrics (does not incur additional costs)
      - BucketSizeBytes
      - NumberOfObjects
    - There are also a number of Request Metrics available. This incurs additional cost.
      - HTTP Operations (GET, POST...)
      - HTTP Error Codes (500, 404...)
      - Total Latency
      - Bytes Uploaded/Bytes Downloaded
    - Cloudwatch Logs integrated with Cloudtrail

### Performance Optimization
  - **S3 Transfer Acceleration:** For uoloads over long distances. [Speed Comparator Online](https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html).
    - Uses Global Edge Locations
    - Uses Amazon Backbone for most of the data transfer
    - Additional charges may apply
  - **Multipart upload API:** FOr uploading large objects
  - **Range GETs:** `GET` performance optimization with CloudFront
  - **S3 Inventory:** For optimizing *list objects and their meta-data* operation instead of using list API
  - **Request Rate Performance(on a single partition):**
    - 3500 `PUT/POST/DELETE` combined requests per second
    - 5500 `GET` requests per second
    - Amazon *automatically* uses multiple partitions if the no. of requests exceed the above limits to avoid seeing `HTTP 500` error codes. You can also do pre-partitioning of data (work with AWS Support)
  - **Parallalization:**
     - Parallal uploads
     - Multiplart uploads (consider this when oject size exceeds 100MB)
     - Multipart downloads (`TransferManager` class of S3 Java SDK)
  - **Amazon S3 Select:** Retrieve only a subset of data based on a SQL expression
    - Lesser data --> Lesser Cost (Pay-as-you-go)
    - Available as API (Like `GET` requests)
    - Used with- AWS Tools and SDKS, AWS CLI and AWS S3 Console (S3 console data access limits to 40 MB)
  - **Amazon CloudFront**
    - TTL (Time-to-live) cache on edge locations
    - Edge Locations are different than AZ and regions
    - You can delete the objects cached in CloudFront, but it will be charged.

## Storage Gateway
- Provides way to access AWS storage infrastructure from on-premises software appliances
- A virtual appliance (available to download virtual machine image) installed on the on-premises as hypervisor 
- Propogates data from your on-premises data center to S3/Glaciar
- Types of storage gateway:
  - **File Gateway (NFS)**
    - For storing flat files only
  - **Volumes Gateway** (iSCSI- block based storage like a virtual hard disk)
    - This is block storage (can be used for OS, applications, etc.), It has two types:
      - Stored Volumes:
        - Data is stored locally (on-prem hard disks) and then backed up into S3 in the form of Amazon EBS (incremental) snapshots asynchronously
        - 1GB to 16TB in size for stored volumes
        - Needs more storage capacity on-prem
      - Cached Volumes:
        - S3 is primary data storage and frequently accessed data is cached locally in your storage gateway providing low-latency access to frequently accessed data from on-prem
        - You can create storage volumes from 1GB to 32TB and attach them as iSCSI devices from on-prem app servers
        - Needs less storage capacity on-prem compared to Stored Volumes
  - **Tape Gateway (Virtual Tape Library- VTL)** - Backup/archival using virtual tapes
    - Cost-effective durable solution for archiving data in AWS

## Snowball
- **Snowball**
  - Petabyte scale data transport hardware appliance to move data in/out from AWS
  - 80TB snowball in all regions and 50TB snowballs in US regions
  - tamper resistant, AES-256 encrypted, full chain of custody
- **Snowball Edge**
  - Snowball like but with 100TB data transfer and on-board storage and compute capability
  - Like small AWS data-center
- **Snowmobile**
  - A shipping container on a truck for 100 Peta Bytes scale data transport for hexabyte-scale data tranfer
- Import and export to/from S3

## Elastic Compute Cloud
- Elastic Compute Cloud (EC2) provides 4 Pricing Options:
1. On Demand
  - Pay per hour of usage without commitment
  - Good for unpredictable load
2. Reserved
  - Term: 1 or 3 years contract
  - Purchase Options: No Upfront, Partial Upfront or All Upfront
  - Offering Class: Standard(Up to 75% off on-demand), Convertible(Up to 54% off on-demand)
  - Scheduled RIs
  - Good for predictable loads
3. Spot
  - Bid on the price
  - Good for apps that are flexible with start and end times
4. Dedicated Hosts
  - Physical servers dedicated for you
  - Use your software licenses on these servers hosted in AWS
  - Good for regulatory requirements which may not support multi-tenancy

## Autoscaling

## Elastic Block Storage
- Think of Elastic Block Storage (EBS) as a virtual disk in AWS cloud. EBS is attached to an EC2 instance.
- EBS Volume Types:
  - General Purpose SSD(GP2)
    - Up to 10000 IOPs
  - Provisioned IOPS SSD(IO1)
    - More than 10000 IOPS upto 25000 IOPS per EBS volume
  - Throughput Optimized HDD(ST1)
    - Can't be a bootable volume
  - Magnetic:
    - Cold HDD
      - Lowest Cost HDD volume. Good for less frequently accessed workloads
      - Can't be a bootable volume
      - Can be used as file server
    - Magnetic (Standard): Previous Generation but still can be used.
      - Can be a boot volume
    - Throuhput Optimized HDD: Low cost HDD. Good for frequently accessed, throughput-intesive workloads
    
- There are many EC2 instance types otimized for specific kind of work loads (e.g. memory optimized, IO optimized, CPU optimized, etc.)

## Elastic Load Balancer
- Three types of Elastic Load Balancers(ELBs):
1. **Application Load Balancer(ALB):** 
  - Works on request level (Application Layer of Open Systems Interconnection(OSI) network model). Handles HTTP/HTTPS traffic optimally.
  - Needs that Security Groups are configured to allow specific traffic to the ALB. You cab use existing SG or create a new SG.
  - **Target Types** can be- 1. **Instance** 2. **IP** or 3. **Lambda Function**
    - Health checks will count as a request for your Lambda function. This means that it will impact cost of lambda function.
  - Health Check Path should be a valid URI(must start with `/`) that points to a resource which exists in the web app e.g. `/index.html` or `/healthcheck.html`

2. **Network Load Balancer(NLB):** 
  - Works on Network Level of OSI model. Handles millions of TCP requests per second securely(TLS) with ultra-high performance.
  - Does not need Security Groups configuration to allow traffic to the NLB itself.
    - The security groups for your instances must allow traffic from the VPC CIDR on the health check port
  - **Target Types** can be- 1. **Instance** or 2. **IP**  
  - Health Check is not a path but it is the port on which traffic is allowed on NLB or can be a different port
  - You may also add one Elastic IP per Availability Zone if you wish to have specific addresses for your load balancer.

3. **Classic Load Balancer(usually referred as ELB):** 
  - For both HTTP/HTTPS and TCP. Choose a Classic Load Balancer when you have an existing application running in the EC2-Classic network.

- For internet-facing load balancers, the IPv4 addresses of the nodes are assigned by AWS. For internal load balancers, the IPv4 addresses are assigned from the subnet CIDR.
- You must select at least two Subnets in different Availability Zones to provide higher availability for your load balancer

## Elastic File Service

## Lambda Functions

## Cloudwatch


## Virtual Private Cloud And Other Services
- You can specify only one subnet per Availability Zone

## Route 53

## Databases

## Simple Queue Service

## Simple Workflow Service

## Simple Notification Service

## Elastic Transcoder

## API Gateway

## Kinesis

## CloudFormation

## Elastic Container Service

## Other Notes

### AWS services that are specific to a region
The below AWS services are specific to a AWS region. E.g. If you plan to launch AWS EC2 instances in multiple regions, you'll need to create a security group in each region:
1. Security Groups
2. IAM Keys

### AWS services that are *NOT* specific to a region
1. IAM Roles
2. [Amazon S3](#simple-storage-service)

## Exam Structure In January 2019
One of my colleagues passed the AWS Certified Solutions Architect Associate Exam in January 2019. Following are the tips from her based on the questions appeared in the exam:
Important Topics:
- S3
- EC2
- VPC
- CloudFormation
- Elastic Beanstalk
- Kinesis
- RDS
- Encryption

**Some questions appeared on AWS Certified Solutions Architect Associate Exam in January 2019:**
1. CloudFormation templates structure in json format was given to check if those are valid templates
2. Which database to use in order for the application to be fully managed, highly scalable, n latency in milliseconds?
3. In order to give end user low latency why do you attach Cloudfront on S3 and why not on EC2?
4. How will you make your EC2 application more disaster recoverable. (copy to another AZ, Region, take snapshots)?
5. How will you encrypt data at rest (SSL,KMS,STS)?
6. How will you encrypt data in transit in S3?
7. How will you encrypt data in S3 if you dont want to manage keys and what will you use if you want to manage keys?
8. Your data is infrequently accessed. What will be the cheapest solution by which you can retrieve data within milliseconds?
9. Database tier needs access to web tier and web tier needs access to internet; what needs to be done?
