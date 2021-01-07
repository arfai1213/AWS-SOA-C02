# AWS-SOA-C01 - AWS Certified SysOps Administrator - Associate
My study guide to study AWS Certified SysOps Adminstrator - Associate

## Useful Links:
<a href="https://d1.awsstatic.com/training-and-certification/docs-sysops-associate/AWS-Certified-SysOps-Administrator-Associate_Exam-Guide_C01.pdf">Offical Exam Guide</a>


<a href="https://d1.awsstatic.com/training-and-certification/docs-sysops-associate/AWS-Certified-SysOps-Administrator-Associate_Sample-Questions_C01.pdf">Offical Sample Exam Questions</a>


## Table of Content:
1. <a href="#cloudwatch">CloudWatch</a>
2. <a href="#ebs">EBS</a>
3. <a href="#elb">ELB</a>
4. <a href="#elasticache">Elasticache</a>
5. <a href="#aws-organizations">AWS Organizations</a>
6. <a href="#ec2">EC2</a>
7. <a href="#aws-config">AWS Config</a>
8. <a href="#bastion-host">Bastion Host</a>
9. <a href="#systems-manager">Systems Manager</a>
10. <a href="#rds">RDS</a>
11. <a href="#aurora">Aurora</a>
12. <a href="#cloudfront">CloudFront</a>
13. <a href="#s3">S3</a>
14. <a href="#kms">kms</a>
14. <a href="#cloudhsm">CloudHSM</a>



## CloudWatch
- Monitoring service to monitor AWS resources.
- By default, Cloudwatch supports montoring on CPU, Network, Disk, Status Check. **RAM Utilization** is a custom metric. 
- Default monitoring interval: 5 mins
- Detailed monitoring interval: down to 1 minute intervals 

### CloudWatch Metrics
- retrieve data using **GetMetricStatistics** API
- Store **as long as you want** (indefinitely)
- Change the retention for each *Log Group*
- able to retrieve data logs from terminated EC2/ELB instances
- Metric Granularity: from 1 minute to 3/5 minutes
- 1 minute for detailed monitoring. 5 minutes for standard monitoring.
- *Minimum granularity* for custom metrics: 1 minutes

### CloudWatch Alarms
- clarm to monitor any CloudWatch metric
- set approriate thresholds to trigger the alarms and related actions

### CloudWatch on-premise
- download and install the SSM agent and CloudWatch agent. 

### CloudWatch Custom Dashboard
- can create CloudWatch metrics (even in different regions) into one place.

### Billing Alarms
- uses SNS topic to send out when the cost reach certain threshold. 

## EBS
- allows to create storage volumes and attach to EC2 instances
- used to create a file system, run a database, etc

Volume Types:
- General Purpose (SSD) - gp2
    - 3 IOPS/GB up to a maximum of 10,000 IOPS
    - System boot volumes
    - low-latency
- Provisioned IOPS (SSD) - io1
    - 50 IOPS/GB to a maximum of 64,000 IOPS
    - I/O intensive, NoSQL / relational databases
    - for critical applications for >10k IOPS / 160MiB/s throughput per volume
    - for large databse workloads
- Throughput Optizmized (HDD) - st1
    - streaming workloads requiring consistent, fast throughput at a low price
    - for big data warehouses and log processing
    - **Cannot be a boot volume**
- Cold (HDD) - sc1
    - legecy
    - for infrequently access
    - lowest storage
    - **Cannot be a boot volume**

### Monitoring EBS
- Pre-Warming EBS Volumes: storage blocks on volumes were restored fomr snapshots must be initialized (pulled down from S3 and writter to the volume) before you can access the block. This preliminary action takes time and cause a significant increase in the latency of an I/O application the first time each block is accessed.
- To avoid this issue: Initialization - reading from all of the blocks on your volume before using it. 

EBS Volume metrics (<a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using_cloudwatch_ebs.html">Link</a> for details):
- `VolumeReadBytes`
- `VolumeWriteBytes`
- `VolumeReadOps`
- `VolumeWriteOps`
- `VolumeTotalReadTime`
- `VolumeTotalWriteTime`
- `VolumeIdleTime`
- `VolumeQueueLength`
- `VolumeThroughputPercentage`
- `VolumeConsumedReadWriteOps`
- `BurstBalance`

EBS Volume statuses: 
<img width="1200" src="images/monitor-ebs-status.png">
- Degraded/Severely Degraded = **Warning**
- Stalled/Not Available = **Impaired** (not available)

Modifiying EBS Volumes:
- can increase EBS size, volume type and adjust IOPS performance (for io1 volume) **without** deteching it.


### EBS vs Instance Store
- EBS:
    - allows data persistency and save data permanently 
- Instance Store: 
    - ephemeral storage which means non-persistence and temporary storage
    - cannot be stopped
    - volumes is lost if failure of underlying drive/ stopping EBS-backed instance/ terminate an instance
- <img width="800" src="images/instance-store-vs-ebs.png">

### Moving EBS to another region/AZ
for moving between AZ:
1. stop EC2 instance (optional but best practice)
2. create snapshot
3. "Actions" => "Create Volume" => choose AZ
    - can change volume type and size.. etc
    - encrpyt

for moving between regions:
1. "Actions" => "Copy" => select region
2. Create Image from EBS Snapshot

### Snapshots
- exist on S3
- point in time copies of Volumes
- incremental
- can take snapshot while the instance is running
- can create AMI's from both images and snapshots
- can change EBS volume sizes on the fly, including chaning the size and storage type
- Volumes will **ALWAYS** be in the same AZ as the EC2 instance

## ELB
- consists of
    - Application Load Balancer (Layer 7)
    - Network Load Balancer (Layer 4, high network thoughtput)
    - Classic Load Balancer

### ELB error messages
- 400 - Bad request
- 401 - Unauthorized
- 403 - Forbidden - blocked by WAF access control list
- 460 - Client closed connection before the load balancer could respond
- 463 - Load balancer received an X-Forwarded-For request header with >30 IP addresses
- 4XX - client side errors

- 500 - Internal server error
- 502 - Bad gateway - application server closed the connection to the load balancer
- 503 - service unavailable - no registered targets 
- 504 - Gateway timeout - e.g. application is not responding
- 561 - Unauthorized - received an error code from the ID provider when trying to authenticate a user
- 5XX - server side errors

### Monitor Load Balancers
1. CloudWatch metrics
    - auto pop-up CloudWatch metrics
    - default: 60sec intervals
    - `BackendConnectionErrors`
    - `HealthyHostCount`
    - `UnHealthyHostCount`
    - `HTTPCode_Backend_2XX,3XX,4XX,5XX`
    - `Latency`
    - `RequestCount`
    - `SurgeQueueLength` (for classic load balancer only) # of pending requests, max queue size: 1024
    - `SpilloverCount` - (for classic load balancer only) # of requests rejected because the surge queue is full

2. Access logs
    - disabled by default
    - capture detailed information about requests sent to load balancer.
    - e.g. time of the request was received, client's IP address, latencies, request paths, and server responses.
    - stores in Amazon S3 bucket
    - **can store data where the EC2 instance has been deleted**

3. Request tracing
    - track HTTP requests from clients to targets or other services
    -  `X-Amzn-Trace-Id` header being added or updated when the load balancer receives a request from client
    - **for application load balancer only**

4. CloudTrail logs
    - CloudTrail monitors API calls in the AWS platform.
    - for audit purpose

## Elasticache
- used to significantly improve latency and throughput for read-heavy application workloads
- improves application performance by storing critical pieces of data in memory for low-latency access

### Memcached
    - does not support multi-AZ

### Redis
    - support multi-AZ
    - in memory key-value store supports data strutures such as sorted sets and lists
    - supports master/slave replication

### Monitoring elasticache
1. CPU Utilization
- (Memcached) add more nodes if exceeds 90% CPU utilization
- (Redis) threshold to deletemine scaling: 90 / # of cores
2. Swap Usage
- Memcached
    - should be around 0 most of the time and should not exceed 50Mb.
    - if swap usage exceeds 50MB, increase the `memcached_connections_overhead` parameters (defines the amount of memory to be reserved for memcached connections and other misc overhead)
- Redis
    - no swap usage metric, use reserved-memory instead
3. Evictions
- Memcached
    - either scale up (increase the memory of existing nodes) / scale out (add more nodes)
- Redis
    - Scale out (add read replicas)
4. Concurrent Connections


## AWS Organizations
- manage multiple AWS accounts at once. 
- allows centrally manage policies across multiple AWS accounts
- allows control access to AWS services
- allows automate AWS account creation and management
- allows consolidate billing across multiple AWS accounts
    - take advantages of pricing benefits from aggregated usage, e.g. volume discounts for EC2 and S3. 

### Service Control Policies
- control AWS service use across multiple AWS accounts.
- specify Allow/Deny individual AWS services

## EC2

### EC2 Options
#### On Demand
- low cost + flexibility without any up-front payment / long-term payment
- unpredictable

#### Reserved
- 1/3 years
- require reserved capacity
- upfront payments
    - Standard RI's (up to 75% off ondemand)
    - Convertible RI's (up to 54% off on demand)
        - capable to change the attributes of the RI to a equal or greater value
    - Scheduled RI's available to launch within the time windows that reserved
        - suitable for predictable recurring schedule

#### Spot
- flexible start and end times
- like stock market

#### Dedicted Hosts
- for regulatory requirements that may not support multi-tenant virtualization
- can be purphased On-Demand / Reservation for up to 70% ooff the on-demand price

### EC2 Launch Issues
#### `InstanceLimitExceeded` error - reached the limitation of the number of instances you can launch in a region. (20 by default)
#### `InsufficientInstanceCapacity` error - AWS does not currently have enough available on-demand capacity to serve your request

### Placement Groups
- By default
    - place instances across different physical hardware
    - minimizes the impact of a hardware failure
    - good for building resilient, highly available systems
    - not so great for low latency, high network throughput applications

#### Cluster
- instances are all created in **same AZ**
- good for requiring high network throughput
#### Partition
- can be multi-AZ
- instances are created in logical segments called partitions
- each located in a separate racks, with independent network and power
- may have multiple instance within the same partition
- great for large distributed / replicated workloads, such as HDFS, HBase and Cassandra
#### Spread
- only have 7 running instances per AZ
- each instance is created in a separate rack, with indepentent network and power
- good for small number of critical instances that should be separated
- reduces the risk of simultaneous failures when instances share the same racks
- guarantees your instances are placed in different racks with isolated power and networking

### AMI
- provides all the information needed to launch an EC2 instance
    - template for root volume, e.g. OS, Applications
    - Launch permissions - defining which AWS accounts can use the AMI to launch instances
    - Block device mapping to specify EBS volumes to attach to the instance at launch time
- registered on a per-region basis
- Sharing AMIs:
    - Copying AMIs: the owner of source AMI must grant read permissions for the storage that backs the AMI
    - Limitation 1: cannot directly copy an encrypted AMI shared by another account
        - copy the snapshot and re-encrypt using your own key
        - the sharing account must share with you the underlying snapshot and encryption key used to create the AMI
    - Limitation 2: You cannot directly copy an AMI with an associated `billingProducts` code (applies to Windows, RedHat and AMIs from AWS Marketplace.)

## AWS Config
- per region basis
- provides AWS resource inventory, configuration history and configuration change notifications to enable security and governance
- Compliance auditing
- Security analysis
- Resource tracking
- provides configuration snapshots and logs config change of AWS resources
- provides automated compliance checking

infomation that we can see:
- Resource Type
- Resource ID
- Compliance
- Timeline
    - Configuration Details
    - Relationships
    - Changes
    - CloudTrail Events

### AWS Config Rules
- Compliance checks:
    - Trigger
        - Periodic
        - Configuration snapshot delivery (filterable)
    - Managed Rules
        - basic

### Configuration **Items**
- Point-in-time attributes of resource

### Configuration **Snapshots**
- Collection of Config Items

### Configuration **Stream**
- Stream of changed Config Items

### Configuration **History**
- Collection of config items for a resource over time

### Configuration **Recorder**
- The configuration of Config that records and stores config items
- Setup:
    - logs config for account in region
    - stores in S3
    - Notify via SNS

## Bastion Host
- a host located in public subnet
- allow to connect to EC2 instance using SSH/RDP
- login to the bastion host over the internet, from your desktop. then you can use the bastion host to initiate an SSH/RDP session over the private subnet to your EC2 instances in the private subnet
- enable safely administer your EC2 instance without exposing them to the internet

## Systems Manager
- gives visibility and control over your AWS infrastructure
- integrates with CloudWatch allowing to view your dashboards, view operational data and detect problems
- includes *Run Command* which automates operational tasks across resources - e.g. security patching, package installs
- organize your inventory, grouping resources together by application or environment - including on-premises systems.

### Run Command
- allow you to run pre-defined commands on 1 or more EC2 instances
- Stop, restart, terminate, re-size instance
- attach / detach EBS volumes
- Create snapshots, backup DynamoDB tables
- Apply patches and updates
- Run an Ansible playbook
- Run a shell script

### AWS Systems Manager Parameter Store
- storing secrets and configuration data management + encrypt

### AWS Systems Manager Activations
- manage hybrid enviroment
- requires ssm-agent installed on-premise

## RDS

Supported versions:
- MySQL
- PostgreSQL
- MariaDB
- Aurora

### Multi-AZ 
- for for disaster recovery
- high availability
- backups/ restore's are taken from secondary which avoids I/O suspension to the primary
- AWS handles the failover. Done by updating the private DNS for the database endpoint
- manually reboot the instance for manually failover

### Read Replicas
- Key metric: **REPLICA LAG**
- can be multi-AZ
- DB Snapshots and automated backups cannot be taken
- `CreateDBInstanceReadReplica` API / AWS management console
- replicated using a supported engine's native, asynchronous replication
- for excess read traffic
- serving read traffic while the source DB instance is unavailable
- for business reporting or data warehousing scenarios
- brief I/O suspension when creating a new read replica (if Multi-AZ is not enabled)
- can be promotedd to it's own standalone database (it will break the asynchronous replication)

### Encrpting RDS Snapshots
Steps:
1. take a snapshot of existing RDS instance
2. copy the snapshot to the region
3. encrypt the copy during the copy process
4. restore the snapshot

### Sharing Encrypted RDS Snapshots between AWS accounts
steps:
1. create a **CUSTOM** KMS Encryption key
2. create an RDS snapshot using the custom key
3. Share the **CUSTOM** AWS KMS encryption key that was used to encrypt the snapshot
4. Use the AWS Management Console, AWS CLI, or Amazon RDS API to share the encrypted snapshot with the other accounts

- **CANNOT** share encrypted snapshots as public
- **CANNOT** share Oracle or Microsoft SQL server snapshots that are encrypted using Transparent Data Encryption (TDE).
- **CANNOT** share a snapshot that has been encrypted using the **default** AWS KMS encryption key of the AWS account that shared the snapshot

## Aurora
- encryption at rest is turned on by default
- MySQL/PostgreSQL compatible, relational database engine that combines the speed and availablity of high-end commercial database
- provides up to five times better performance than MySQL (and three times better performance than PostgreSQL) at a price point one tenth that of a commercial database while delivering similar performancee and availablity
- Start with 10GB, Scales in 10GB increments to 64TB (Storage Autoscaling)
- compute resources can scale up to 64vCPUs and 488GiB of memory
- **2 copies of data is contained in each AZ, with minimum of 3 AZ. 6 copies of your data.**
- Aurora storage is self-healing, Data blocks and disk are continuouly scanned for errors and repaired automatically
- 100% CPU Utilization? Scale Up if writes causing the isses. Scale Out read replicas if reads causing the issues 
- Failover is defined by tiers. The lower tier the higher priority being the highest priority available (Tier 0)

### Cross Region Replicas
- creating a new cross region replica will also create a new Aurora cluster in the target region.
- If the replication is disrupted, you will have to set up again
- **Recommened** to select `Multi-AZ Deployment` to ensure high avilability for the target cluster

### Aurora Serverless
- on-demand, auto-scaling configuration for Aurora where the database will automatically start-up, shutdown and scale up or down capacity based on your application's needs.
- pay on a per-second basis for the database capacity you use when the database is active


## CloudFront
- deliver webpages and other web content to a user based on the geographic locations of the user, the orgin of the webpage and a content delivery server
- Edge Location - location where content will be cached
- Origin - the origin of all the files that the CDN will distribute
- Distribution - the name given the CDN which consists of a collection of Edge Locations

- Web Distribution - used for websites
- RTMP - used for Media Streaming

### Cache Hit Ratios
- the ratio of request served from edge locations
- more request from edge location, the better the performance
- Maximise Cache Hit Ratio:
    - specify how long cloudfront caches your object
        - `Cache-Control` and `max-age`
    - caching based on query string parameters (*case sensitive)
    - caching based on cookie values
    - caching based on request headers
    - remove `Accept-Encoding` header when compression is not needed
    - serving media content by using http

## S3
- object-based
- size from 0 Bytes to 5TB
- unlimited storage
- Read after Write consistency for PUTS of new Objects
- Eventual Consistency for overwrite PUTS and DELETES (may take some time to propagate)

### S3 Storage Tiers:
- Standard
    - designed to sustain the loss of 2 facilities concurrently
- Standard - IA
    - for data that is accessed less frequently, but requires rapid access when needed
    - lower fee than S3 but charged for a retrieval fee
- Standard - One Zone IA
    - same as S3-IA but in one AZ only
    - cost is 20% less than S3-IA
- Reduced Redundancy Storage
    - used for data that can be recreated if lost, e.g. thumbnails.
- Glacier
    - very cheap, used for archival only
    - takes 3-5 hours to restore
    - **NO REAL TIME ACCESS**

### S3 Intelligent Tiering
- 2 tiers: Frequent and Infrequent access
- for unknown or unpredictable access patterns
- auto. move your data to msot cost-effective tier based on how frequently you access each object
- for cost optimization
- no fees for accessing data but small monthly fee for monitoring / automation

### S3 Charges
- storage per GB
- requests (get, put, copy, etc..)
- Storage Management Pricing (Inventory, Analytics and Obejct Tags)
- Data Management Pricing
    - data transfered out of S3
- Transfer Acceleration
    - use of CloudFront to optimize transfers


### S3 Lifecycle Policies
- manage object so that they are stored using the most cost effective S3 option throughout their lifecycle
- e.g. transition objects to less expensive storage classes, archive them or delete them
- e.g. transition objects to an IA storage class 90 days after creation
- e.g. configure objects to expire 1 year after creation

### S3 Versioning
- enables to revert to older version of S3 objects
- multiple version of an object are stored in the same bucket
- with versioning enabled, a **DELETE** action doesn't delete the object version, but applies a**delete marker** instead
- to permanently delete, provide the object Version ID in the delete request

### S3 MFA Delete
- provides an additional layer of protection to S3 Versioning
- enforce 2 things:
    - need a valid code from MFA device in order to permanently delete an object version
    - MFA also needed to suspend / reactivate versioning on an S3 bucket

### S3 Encryption
- In Transit: SSL/TLS
- At Rest:
    - Server Side Encryption:
        - S3 Managed key: **SSE-S3**
        - AWS KMS, managed keys, **SSE-KMS**
        - Server Side Encryption with Custom Provided Keys - **SSE-C**
            - customer in charge of rotation the keys
            - AWS handles the encryption / decryption
    - Client Side Encryption
        - encrypt the file before uploading to S3

#### Enforcing encryption on S3 Buckets:
- enforce SSE by using bucket policy which denies any S3 PUT request which doesn't include the  `x-amz-server-side-encryption` parameter in the request header
- includes `x-amz-server-side-encryption` in the request header
    - options:
        1. `x-amz-server-side-encryption`: `AES256` (SSE-S3)
        1. `x-amz-server-side-encryption`: `ams:kms` (SSE-KMS)

## KMS
- shared hardware, multi-tenant managed service
- allow to generate, store and manage your encryption keys
- suitable for applications for which multi-tenancy is not an issue
- encrypt data stored in AWS
- uses symmetric keys

## CloudHSM
- HSMs (Hardware Security Modules) are used to protect the confidentiality of your keys
- dedicated HSM instance, hardware is not shared with otehr tenants
- no free tier
- allow to generate, store and manage your encryption keys
- under your exclusive control within your own VPC
- FIPS 140-2 Level 3 compliance (US Government standard for HSMs)
    - includes tamper-evident physical security mechanisms
- suitable for applications which have a regulatory requirement for dedicated hardware managing cryptographic keys
- symmetric / asymetric encryption