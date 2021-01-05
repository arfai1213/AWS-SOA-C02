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
- each instance is created in a separate rack, with indepentent network and power
- good for small number of critical instances that should be separated
- reduces the risk of simultaneous failures when instances share the same racks
- guarantees your instances are placed in different racks with isolated power and networking



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