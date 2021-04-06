# AWS Solution Architect Associate

## IAM

* __Roles__

* __Groups__

* __Resources__
  * Prevent users to access non-authorized resources

### Hints

* Always delete Root Access keys and enable MFA for root user
* Never use root-user

## EC2

= Elastic Cloud Computing

### SSH to EC2 machines

* Access keys are defined during machine set-up
* Make sure to run `CHMOD 0400 file-name.pem` on your machine to enable access to private key, but protect against misusage
* Invoke particular key file usage during SSH login: `ssh -i "/path/to/key-file.pem"`
* __SSH Browser Connect__ is AWS utility to connect from browser session; It temporarly upload Access Key to machine enabling connection; Ingress port `22` needs to be open in __Security Group__

## Security Groups

* It's stateful security measurement comparing to ACL, which is stateless measurement. (Steful means that once connection is established, security rules are not evaluated each time. E.g. if there is inboud traffic allows for http/8080, also response request is allowed even if not allowed by rules)

* Security Groups are defined within Region/VPC combination

* Best practise is to separate dedicated security group for SSH only

* Security group blocking access leads to to __timeouts__; `Connection refused` is usually caused by application/server issue

* You can authorize incoming access based on client IP/port, but also based on Security Group assigned to client machine within same VPC. e.g. all machines with assigned Security Group XXX, can access my server

## Networking

### Public, Private IP

* __Elastic IP__ is static Public IP; It's needed to be released back to pool from Networking/Elastic IP menu once it's not associated to any machine, otherwise it's still charged

## EC2 instancies

__Instance Types__:
1. __M__ "Medium" General Purpose Instances
2. __C__ "CPU" Compute Optimized Instances
3. __R__ "RAM" Memory Optimized Instances
4. __G__ "GPU" Accelerated Computing Instances
6. __I__ "IOPs" Storage Optimized Instances
7. __T__ "BustT" Burstable general purpose instancies
  * Burst up to capacity
  * Umlimited burst
  * Burst credits are obtained during low performance periods and consumed during high performance periods

__Instance Launch Types__:
1. On-Demand
2. Reserved Instances
3. Convertible Reserved Instance
4. Scheduled Reserved
5. Spot Instances
   * Spot requests - One-Time / Persistent
   * Spot Fleets  
     * enable to define resource pools across AZ, OS, instance Types
     * Optimizing strategies: LowestCosts, Diverse, CapacityOptimized
7. Dedicated Hosts - with access to cores, etc. (e.g. in case you need Cores for licensing puroposes)
8. Dedicated Instances

## AMI = Amazon Machine Image

* AMI is region dependent
* AMI can be public - for free or for rent via AWS Marketplace
* AMI is persisteted in S3
* AMI are created from existing instance 

### Cross Account AMI Copy

* You can share AMI with another AWS Account, and you are still the owner
* If you copy AMi shared with you and you will become the owner of target AMI
* __billingProduct__ = includes Windows AMI or AMI from AWS Marketplace
* Any AMI (including AWS Marketplace) can be copied by launching an new EC2 instance and creating new image from the instance
  * Are such a images also billed? Probably so...

### Placement Group

* __Cluster__:
  * All instancies are on the same rack (means same HW, also same AZ)
  * Very ow latency
  * No HA!!!
  * Available only for expensive EC2 instancies
*  __Spread__:
  * Each instantce on different HW, spread accross multiple AZ
  * Delivers HA
  * Max 7 istancies per AZ per Placement Group
*  __Partition__:
  * Partition = set of racks
  * You can create up to 7 partitions
  * EC2 instancies access metadata - to which partition they belong to

### ENI - Elastic Ntwork Interfaces

* = Virtual Network Card
* Each have: Private IPv4, One Elastic IP per private IPv4, associated security groups, MAC Address
* You can create and re-attach (secondary IPs) them on the fly (for failover)
* It's AZ dependent (you can pair only with EC2 within same AZ)

### EC2 Hibernate

* -> RAM of EC2 is dumped to EBS root memory (encryption required) and reloaded during restart -> no real stop/restart of OS
