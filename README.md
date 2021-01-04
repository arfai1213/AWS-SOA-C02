# AWS-SOA-C01 - AWS Certified SysOps Administrator - Associate
My study guide to study AWS Certified SysOps Adminstrator - Associate

## Useful Links:
<a href="https://d1.awsstatic.com/training-and-certification/docs-sysops-associate/AWS-Certified-SysOps-Administrator-Associate_Exam-Guide_C01.pdf">Offical Exam Guide</a>


<a href="https://d1.awsstatic.com/training-and-certification/docs-sysops-associate/AWS-Certified-SysOps-Administrator-Associate_Sample-Questions_C01.pdf">Offical Sample Exam Questions</a>


## Table of Content:
1. <a href="#cloudwatch">CloudWatch</a>
2. <a href="#ebs">EBS</a>



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

## EBS
Volume Types:
- General Purpose (SSD) - gp2
    - burst up to 10,000 IOPS
    - System boot volumes
    - low-latency
- Provisioned IOPS (SSD) - io1
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