# AWS Solutions Architect February 2018 Associate Level Certification Exam Notes

## Contents
* [Identity & Access Management (IAM)](#identity-&-access-management)
* [Simple Storage Service (S3)](#simple-storage-service)
* [Elastic Compute Cloud (EC2)](#elastic-compute-cloud)

## Identity & Access Management
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

## Simple Storage Service
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

- Bucket Policies are similar to IAM policy but are applied to AWS resources (S3 in this case). Hence it will also have *principal* defined in the policy as opposed to IAM policies. 
- Bucket policies can also have *conditions*. Condition values can be date, time, ARN of requester, IP of requester, user name, user id and user agent. S3 policy also support conditions using `object tags`.
- Sample bucket policy:

- Can be used to put size limit policies (upto 20KBs) on S3 buckets/objects.

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

- Access Control Lists: You may not need IAM policies if ACL are sufficient to control access to buckets.

- Predefined Groups:
  - Authenticated Users Group: Represents all AWS accounts **worldwide** and not only authenticated users from your AWS account. **USE WITH CAUTION**
  - All Users Group: Open access to all. Requests can be made by authenticated users or anonymous. **USE WITH CAUTION**
  - Log Delivery Group: Use WRITE permission to this group to allow writing server access logs.

> **NOTE:** Use VPC endpoint for secured connection between EC2 instances and S3. Using VPC endpoint, the traffic from EC2 to S3 bucket will not be directed via internet, making it more secured. You can control access to S3 via VPC endpoint by applying VPC endpoint policies OR using bucket policies. VPC endpoint policy is a `resource policy` which means it needs `principal` to be specified.

### Encryption
#### Data At Rest
##### Server Side Encryption
- AWS provided keys (SSE-S3)
  - Every object is encrypted with a unique key using AES-256 encryption standard. Each unique key is encrypted with a regularly rotating master key
  - Enable `Default Encryption` on bucket to enable encryption or use a bucket policy to enable encryption on all the objects stored in a bucket. The `x-amz-server-side-encryption` can be used in bucket policy to `deny` upload to the bucket if the request does not contain `x-amz-server-side-encryption: AES256`
  
  - 
- AWS KMS managed keys (SSE-KMS)
- Customer provided keys (SSE-C)

##### Client Side Encryption
Encrypt data before uploading to S3

## Elastic Compute Cloud
### AWS services that are specific to a region
The below AWS services are specific to a AWS region. E.g. If you plan to launch AWS EC2 instances in multiple regions, you'll need to create a security group in each region.

1. Security Groups
2. IAM Keys
