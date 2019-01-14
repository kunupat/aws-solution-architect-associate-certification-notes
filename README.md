# AWS Solution Architect February 2018 Associate Level Certification Exam Notes

## IAM

## S3
### S3 Usage Patterns (When to use S3)
1. Store and distribute static web content and media
2. Host Entire Static Web Sites
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
* requests (per thousand requests per month).
## EC2

## AWS services that are specific to a region
The below AWS services are specific to a AWS region. E.g. If you plan to launch AWS EC2 instances in multiple regions, you'll need to create a security group in each region.

1. Security Groups
2. IAM Keys
3. 
