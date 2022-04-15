---
title:  "AWS"
mathjax: true
layout: post
categories: [AWS]
date: 2020-06-13 11:04 +0200
---

# 1.Storage

# AWS storage services

Data storage categorization

- Block storage
- File storage
- Object storage

Block storage:

- Stores data in chunks known as blocks
- Blocks are stored on a volume and attached to a single instance
- They generally provide very low latency data access
- Comparable to DAS storage used on premises

File storage:

- Data is stored as operate files with a series of directories
- Data is stored within a file system
- Provide shared access for multiple users
- Comparable to NAS storage used on premises

Object storage:

- Objects are stored across a flat address space
- Objects are referenced by a unique key
- Each object can also have associated metadata to help categorize and identify the object

Amazon s3

- Fully managed object based storage service
- Highly available and highly durable
- Very cost effective
- Widely and easily available
- Unlimited storage capability
- Highly scalable
- Smallest file size supported - 0 bytes
- Largest file supported - 5 TB

When data is being uploaded to s3 we as the customer need to specify the region for that object. By choosing the region, Amazon will keep duplicate of that object in multiple AZ’s within that region for durability and availability.

Durability and availability in S3

- Objects stored in s3 have a durability of 99.999999999%
- S3 copies numeros copies of the same data in different AZ’s
- The availability of the S3 objects is 99.99%

**Availability** means AWS ensures that the uptime of amazon is 99.99%.

**Durability**  means the probability of maintaining your data without it being lost.

S3 Buckets

- To store objects in s3, first we need to create and define a bucket. Bucket can be thought of like a parent folder for your data.
- Bucket names must be globally unique. Not only region wise but globally unique with the other S3 buckets that exist. Once we create the bucket name we can begin uploading the data.
- Data can be uploaded into bucket or folders within the bucket
- Limitation of 100 buckets per aws account
- Objects that stored in these buckets have unique object key

Storage Classes

- Standard
- Standard - IA (infrequent Access)
- Intelligent Tiering
- One Zone - IA (Infrequent Access)
- Reduced Redundancy Storage(RRS)

Data that is being accessed frequently:

- Standard - > Default storage class -> more cost effective and offer more availability
- Reduced Redundancy Storage(No longer recommended by AWS)

Data that is being accessed infrequently:

- Standard IA and One Zone - IA
- Offers the same access speed to that of Standard
- Additional cost to retrieve data
- For long lived data objects
- One Zone - IA does not replicate data across multiple AZ’s - 99.5 availability

Intelligence Tiering:

- Used for unpredictable access patterns
- Consist 2 tiers: Frequently accessed and infrequently accessed
- Automatically moves data into appropriate tier based on access platform
- Objects must be larger than 128KB

Before choosing storage classes:

- How often is the data likely to be accessed?
- How critical is my data?
- How reproducible is the data?
- Can it be easily created again if need be?
- Do I know the access patterns of my data?

S3 Security

- Bucket policies
    - Allow you to impose set access controls within a specific bucket -> who or what can access data within specific bucket
    - Constructed as JSON policies
    - Only control access to the data in the associated bucket
    - Bucket permissions can be very specific using policy conditions -> like specific time range and specific IP
    - Provide added granularity to bucket access
- Access Control List
    - ACL only control access to users outside of your own AWS account, such as public access or access from another aws account
    - Not granular as bucket policies
    - The permissions are broad in access, for example ‘List objects’ and ‘Write objects’
- Data encryption
    - Server side and client side encryption’
    - SSE-S3 (S3 managed Keys)
    - SSE-KMS(KMS managed keys)
    - SSE-C (Customer managed keys)
    - CSE-KMS(KMS managed keys)
    - CSE-C(customer managed keys)
    - SSL is supported

Data management

Versioning:

- Allows for multiple versions of the same object to exist.
- Useful to recover from accidental delete or malicious activity
- Only the latest version of the object is displayed by default
- Versioning is not enabled by default
- If versioning is enabled :
    - You can't disable versioning, only suspend
    - Added cost for storing multiple version of the same object

Lifecycle rule:

- Provides an automatic method of managing the lifecycle of your data stored
- Ability to configure specific criteria to automatically move data between storage classes, including glaciers or even delete the data completely
    - Cost effective exercise
    - Scenario -> move objects to cheaper storage classes after a set period of time
        - Or keep the data for set period of time before deletion
- Time frame is configurable allowing you to set your own requirements

Common Use cases of S3

- Data backup
    - For on premise data or for existing aws services that are using
    - Offers service to backup on premise data
- Static content and website
    - Can access using unique url
    - Cloudfront
    - Whole static website can be hosted
- Large data sets
- 

Limitation of S3

- Data archiving for long term use
- Dynamic and fast changing data
- File system requirements
- Structured data with queries

# Amazon Glacier

- A very low cost, long term, durable storage solution suited for long term backup and archival requirements
- It does not provide instant access to your data
- 99.999999999% durability -> replicate data across multiple AZ’s within a single region
- Storage costs are considerably lower than S3
- Retrieval of your data can take up to several hours

There are no bucket concept in Glacier only valut and archives

Glacier vault:

- :vault acts as a container for glacier archives
- Vaults are regional -> region need to be specified at the time of creation
- Within each vault you can store data as archives

Glacier archives :

- Archives can be any objects similar to s3
- You can have unlimited archives within your glacier vaults

Glacier dashboard

- The glacier dashboard only allows your to create vaults
- Any operational process to upload or retrieve data HAS to be done using code:
    - Amazon SDK
    - The Glacier web service API

Moving data to glacier

- Create your vault
- Move the data in to the vault using API/SDK/
    - If data already exists in S3 -> data can be moved using S3 Lifecycle rule

Data retrieval

- Expidited
    - Used for urgent access to as subset of an archive
    - Less than 250 MB
    - Data available within 1-5 min
- Standard
    - Used to retrieve any of your archives, regarding of size
    - Data available within 3-5 hours
- Bulk
    - Used to retrieve petabytes of data
    - Data available within 5-12 hours
    - The cheapest option for data retrieval

Glacier Security

- Data is encrypted by default using the AES-256 encryption algorithm
- Vault access policy
    - Resource based policy- applied directly to your resource - similar to bucket policy
    - Applied to specific vault
    - Each vault can only contain 1 vault access policy
    - Policy is in json format
    - Policy contains a principal component
- Vault lock policies
    - Once set they cannot be changed
    - Used to help maintain specific governance and compliance controls

# EC2 instance level storage

- Storage resides on the same ec2 host machine
- Provides temporary storage
- Not recommended to store critical or valuable data
- If your instance is stopped or terminated your data is lost
- If rebooted, your data remain in tact

Benefits

- No additional cost for storage, it’s included in the price of the instance
- Offer a very high I/O speed
- Instance store volumes are ideal for cache of buffer for rapidly changing data without the need for retention
- Often used within a load balancing group, where the data is replicated and pooled between the fleet

Delimits:

- Data that need to be persistent
- Data that need to accessed and shared by multiple entities

# Elastic Store Block

- Provides block level storage to EC2 instance
- Offers persistent and durable storage
- Greater flexibility than that of instance storage volumes
- EBS volumes can be attached to EC2 instances for rapidly changing data
- Used to retain valuable data due to its persistence qualities
- Operates as a separate service to EC2
- EBS volumes act as network attached storage services
- Each volumes can only be attached to one EC2 instances
- Multiple EBS volumes can be attached to a single EC2 instances
- Data is retained if the EC2 instance is stopped, restarted or terminated
- EBS volumes can be only attached to EC2 instances that exist in the same within the same AZ

EBS Snapshots

EBS volume => point in time snapshots [manual or automatic schduled] => EBS snapshot[are stored in S3]

- Snapshots are incremental -> only diff of new objects will be snapshoted
- Can create new EBS volume

High availability

- AWS will replicate the EBS volumes in same AZ’s in the same region
- 

EBS volume type

- SSD
    - Suited for work with smaller blocks
    - Such as DB’s using transactional workloads
    - Often used for boot volumes on EC2 instances
    - Two categories
        - General purpose SSD (GP2)
        - Provisioned IOPS (IO1)
- HDD
    - Designs for workload requiring a high rate of throughput
    - Such as big data processing and logging information
    - Larger blocks of data
    - Two categories:
        - Cold HDD(SC1)
        - Throughput Optimized HDD(ST1)



Encryption

- EBS offers encryption at rest and in transit
- Encryption is managed by the EBS service itself
- It can be enabled with a checkbox
- AES 256 -> AWS KMS -> AWS KMS CMK/DMK

Creating a new EBS volume

There are 2 ways to creates:

- During the creation of a new ec2 instance
- As a stand alone EBS volume

Changing the size of an EBS volume

- EBS volumes are elastically scalable
- Increases the volume size via the AWS console or CLI
- After the increase you must extend the filesystem on the ec2 instance to utilize the additional storage
- It is possible to resize a volume by creating a new volume from a snapshot

Pricing - > per month

Disadvantages

- EBS is not well suited for all storage requirements like temporary storage, multi-instance storage access or high durability and high availability
- 

# Elastic File System

- EFS provide a file level storage service
- EFS is fully managed
- Highly available & durable
- Ability to create shared file system
- Highly scalable
- Concurrent access by 1000’s of instances
- Limitless capacity - similar to S3, elastically increase the storage capacity on the requests

Creating EFS file system

- Create EFS from console
- Choose VPC
- EFS will automatically create mount target to you
- Mount target will help you to connect to the EFS from the EC2 instances
- Supported only NFS V4.0, and V4.1
- Does not support Windows
- Linux instance must have the NFS client installed to mount the target
- Next step is to set the performance mode and encryption settings
- Two performance mode
    - General purpose
        - Used in most use cases
        - Provides lowest latency
        - Max of 7000 file system operation per second for your EFS
    - Max I/O
        - Used for huge scale architectures
        - Concurrent access of 1000’s instances
        - Can exceed 7000 file system operations per second
    - 

# Cloudfront

- Acts as a Content Delivery Network
- Distributes data requested through web traffic closer to the end user via edge locations as cache data
- As the data is cached, durability of the data is not possible
- Origin data can come from S3

Edge location

- AWS edge locations are sites deployed in highly populated areas across the globe.
- Edge locations are not used to deploy infrastructure (ECS/EBS)
- Edge locations allow the ability to cache data and reduce latency for end user access with services such as Amazon Cloudfront

Distribution

- Web Distribution
    - Used to distribute static and dynamic content
    - Uses both HTTP and HTTPS protocol
    - Allows you to add, remove and update objects
    - Ability to provide live stream functionality on your website
    - Uses an `origin` to define where the source data is coming from
    - Origins can be a web server , EC2 instance or an Amazon s3 bucket
- RTMP distribution
    - Should be used if your focus is to distribute streaming media using Adobe Flash media server’s RTMP protocol.
    - Allows the end user to start viewing media before the complete file has been downloaded from the edge location.

Distribution Configuration

- You must specify your origin location
- Select specific caching behavioral options
- Define the distribution settings(which edge locations you want your data to be distributed to which can be [US, canada, europe], [US, canada, europe, asia], [All edge locations])


# Storage gateway


- Gateway between your own premises data center and AWS
- The software appliance can be download from aws as a virtual machine which then can be installed in your vm or hypervisor
- Offer 3 configuration
    - File gateway
        - Allow you to securely store files as objects within S3
        - Ability to mount or map drives to an S3 bucket as if it was a share held locally
    - Volume gateaway
        - 
    - Tape Gateway

Complete it tomorrow

# Snowball

- Used for securely transfer large amounts of data in and out of AWS (Petabytes scale)
- Either from your on-premise data center to amazon S3, or from Amazon S3 back to your data center using a physical appliance, known as a snowball
- The snowball appliance comes as either a 50TB or 80 TB device.
- The snowball appliance is dust, water and tamper resistant.

Encryption and Tracking

- By default snowball appliance is automatically encrypted 2 way encryption using KMS.
- Allows gives end to end tracking using a E INK shipping label. This ensures when the device leaves your premises it is sent to the right AWS facility
- The appliance can be tracked using SNS text messages or via the AWS management Console.
- Data removal from the appliance is the responsibility of the AWS , confirming NIST standards.

If your data retrieval takes longer than a week using your existing connection method, then consider using AWS snowball.

# 2.Compute

What is compute

- Brains and processing power required by the application and the system to carry out computational tasks via a series of instructions.

# EC2

- Amazon Machine Images
- Instance types
- Instance purchase options
- Tenancy
- User data
- Storage options
- Security

AMI

- Preconfigured templates for ec2 instances helps you to launch new instances quickly
- AMI is a image baseline which include OS with applications with any custom configurations.
- We can create our own AMI
- AMI market place is place to purchase AMI from trusted ventors
- There are community AMI’s too

Instance type

- Defines size of the instance based on different parameters
- AVX for audio video or scientic analysis
- Vcpus, memory, instance storage, network performance while you create a new instance
- Instances types
    - Micros instances
    - General pupose - small to medium db, testing , backend
    - Compute optimized- high performance service, web service ,
    - GPU instance-
    - FPGA instance - financial computing
    - Memory optimized - realtime processing of unstructured data like microsoft share point
    - Storage optimized

Instance purchasing options

- On-demand instances
- Reserverd
- Scheduled
- Spot
- On-demand capacity reservations

# Cost management

Total Cost Ownership calculator (TCO)

- Free service to compare dc vs aws cloud
- For those who wish to migrate to aws
- How it works
    - Questionnaire
        - Number/types of servers
        - type/amount of storage
        - Option to include details on n/w hardware /labour costs
    - TCO will give a report
        - Saving estimate
        - Cost comparison
        - Show which all aws services is need to mirror dc

Billing dashboard

- Dynamic graphs
- Monthly account cost/usage to date
- Previous month cost /usage
- There are 3 different graphs
    - Spend summary graph
        - Previous month total spent
        - Current month spent month-to-date
        - Estimate total you will spend by month end
    - Spend by Service & Service by Spend
        - Current month-to-date
        - What service are used most
        - Spend by service: percentage of total cost
        - Service by spend: amounts spent on service

AWS cost explorer

- At a glance graph related to spending /usage
- Different from billing dashboard
- It shows 12 months of data
- Forecasts up to 3 months
- Reserved instance recommendations
- Includes preset filters
    - Review data based on aws service, regions, linked a/c, az, instance type, usage type
- Use cases;
    - Graph of cost/ usage data
    - Quick analysis - identify trends, traffic spikes
    - Forecast spendings based on historical data
    - Best for stable usage of services
- Limitations
    - Forecast based on historic data
    - Pre-set graphs and filters
    - Customization not possible

Aws budgets

- Set service cost/usage thresholds for an aws service
- Alert when you exceed thresholds
- There are 3 types of budgets:
    - Cost
    - Usage
    - Reserved instance budgets

Consolidated billing

- For companies that has numerous or separate aws a/cs
- Simplify billing with potential cost savings

Aws customer support

- AWS Customer support plans
    - Basic
    - Developer
    - Business
    - Enterprise


# AWS Trusted advisor

- For optimizing your infra
- Recommends improvements based on best practices
    - What recommendations
        - Cost optimization
        - Performance
        - Security
        - Fault tolerance
- Based on checks
- All checks availble for business/enterprise
- Outside support plan will have access to 6 core checks
    - 6 core checks are split in to performance and security
        - Performance
            - Service limits
        - Security
            - Securiyt groups
            - Ebs public snapshot
            - Rds public snapcshot
            - IAM use
            - MFA on root
- We can see the checks and take actions
- Can restrict to security or cost etc.using IAM
- Can restrict access specific checks using IAM
- cloud watch events + SNS to sent alert on changes

# AWS Config(pending)

- Resourcce management
    - What resources do we have
    - Are there any sec vuln
    - Dependency of resources
- What does Aws config do
    - Captures resource changes
    - Act as an resource inventory
    - Store configuration history
    - Provide snapshot of configurations
    - Nofications about changes
    - Provide cloutrail intergration => who made changes at what time and with which api
    - Use rules to compliancy
    - Security analysis
    - Identity relationships
- Key components
    - Aws resources
        - Objects that can be created, updated, delete from aws management console

# AWS Cloudtrail

- Records and tracks all the **api request** in your AWS account
    - These api calls can from SDK’s, AWS CLI, AWS management console, Another AWS service
- Every api call is recorded as an event
- Multiple events are recorded in cloudtrail logs
- Meta data -> identity of the caller, timestamp, source ip

# AWS cloudwatch

# AWS ECS

- Different ways to set up docker in aws
    - Docker out of the box + vanilla ec2 instance < IasS
    - Aws elastic beanstalk + docker container format <- PaaS
    - Docker datacenter DDC for aws <= CaaS
    - Docker for AWS
- ALB, for prod
- EBS <- persistant storage
- IAM <- learn about different policies

# AWS Elastic Beanstalk

- Upload web app code with env config
- Supports packer builder, singel container docker, multi-container docker, preconfigured docker , almost all web app languages
- No cost
- Cost will be on ec2 or any other service is needed