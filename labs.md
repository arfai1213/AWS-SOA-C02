# Labs
Records what to study in labs

## Table of content:
1. AWS Config
1. VPC
    - setup public, private subnet, internet gatway from scratch
    - NAT gateway to allow instance in private subnet to patch software etc..
    - how NACLs work
    - VPC endpoints
        - Note: `aws s3 ls --region YOUR_REGION` The `--region` flag is needed for calling VPC endpoint
    - create flow log
        - format: `${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}`
1. Load balancing
    - launch configurations, target groups, ASG
1. AWS Organizations
    - how to use SCP to control service access
    - how the policy inheritance works