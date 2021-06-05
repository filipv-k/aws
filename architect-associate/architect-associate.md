# AWS Solution Architect Associate

## IAM

* __Roles__

* __Groups__

* __Resources__
  * Prevent users to access non-authorized resources

* __AWS Policy Simulator__ - test your settings

### Hints

* Always delete Root Access keys and enable MFA for root user
* Never use root-user

## EC2

* = Elastic Cloud Computing
* __EC2 Instance Metadata__: Learn about themsevels without using IAM role
  * from internal IP: <http://169.254.169.254/latest/meta-data/> (but not any IAM role)

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

## HA, Scaleability, ELB, Autoscaling Groups

## Connection Draining

* Called __Connection Draining__ for CLB
* Called __Deregistration Delay__ for ALB, NLB
* == Delay between moment, when LB stop to sent traffic to EC2 and final termination of EC2 == time to complete pending requests
* Default = 300s, possible internaval 1 - 3600; deactivated by 0

## Autoscaling Groups

* scalling out/scaling in workload
* possible to set minimum/maximum workload
* Integration with NLB
* Free of charge (paid by resources)
* IAM attached to ASG is asigned to EC2
* AGS terminates unhealthy EC2 (clasified by LB)
* __Launch Configuration__:
  * AMI + Instance Type
  * ...
  * New version is __Launch Templates__
* __Alarms__

  * Based on CloudWatch Alarm (based on monitoring some metrics - average CPU, # request, Network in, netwrok out, etc. )
  * Metrics can be defined on *schedule*
  * Metrics are calculated as average per autoscaling group
  * __Auto Scaling Custom Metric__ - calculated inside app, passed via API PutMetric to CloudWatch

* Scaling Policies

  * __Target Tracking Scaling__ - targeting to get close to given goal, e.g. CPU = 70 %
  * __Simple/Step Scaling__ - define sigle decisison to take action, you need to have set of these rules
  * __Scheduled Actions__ - action is taken on given time (eg. Monday morning)
  * __Scaling Cooldowns__ - time before next scaling policy action takes place; there is default value, which can be overwritten by simple step

* __Lifecycle Hooks__ - Additional steps to install/ceck before going to status InService or to Terminate

## RDS - Relational Database Service

* RDS can be set as encrypted only during creations; there is workaround via encrypting the snapshot and restoring db from encrypted snapshot

### Read Replicas

* ASYNC replication between Master and Read Replica (Eventual Consistency)
* Replica can be promoted to separate independent DB (no connection to original master)
* __Network Costs Exeption__ - No fees for replication between AZ within same region
* There is NO network replication between regions

### RDS Multi-AZ (Disater Recovery)

* SYNC replication between 2 instancies in 2 AZ
* Behind single DNS entry
* Automatic fail-over to switch
* Read Replicas can be also used as Multi AZ for Disaster Recovery (but there is ASYNC replication -> some data loss may occurs)
* Multi-AZ setup is Zero Downtime operation (it's replicated via Snapshot, with some performance penalty)
* RDS can be __at rest__ / __in flight__ encrypted
* __at rest__ Master can be encrypted, replica can be encrypted only if master is encrypted

### RDS Security

* Deploy in private subnet
* Attach SG to control access
* AWS IAM controls only RDS management actions (create db, create snaphosts, etc.) not domain auth, as these are controlled by IAM of DB (only for MySQL and PostreSQL you can use IAM authentication)
  * Access token is provided by RDS service (Probably some SAML integration between RDS service and RDS DB instance??? - NO, it uses AWS Signiture)
  * This requires SSL connection to RDS DB (Why? we are already in SSL termination phase)

## Aurora DB

* It's not part of RDS
* AWS proprietary, not open sourced
* MySQL, PostreSQL compatible - you always have to chose one of these 2 PostreSQL/MySQL 
* Performance optimalizations against MySQL, PostreSQL
* Autometical auto-scaling by 10GB up to 64TB
* Native HA - 6 copies across 3 AZ
* Failover in 30 seconds
* 1 Master, max 15 replicas
* Cross region replication
* Writer Endpoint - DNS always to Master
* Reader Endpoint - Loadbalancing on the top of read replicas
* Backtrack - restoring requested status without having backups

## Others

* IAM can be used to connect to database (default alternative is standard user management)
* Encryption at rest
  * (not moving data) with KMS - AWS-256 encryption
  * Has to be difined at launch time
  * Replicas can be entcrypted only if Master is encrypted
* in-flight encryption
  * SSL connection
  * Enforce SSL via DB specific statements: PostgreSQL - parameter group, MySQL - admin SQL statement

## Route 53

* Type of records: __A__:hostname to IPv4, __AAAA__: hostname to IPv6, __CNAME__: hostname to hostname (even not AWS hostname!), __Alias__: hostname to AWS resource
* It's GLOBAL service, no region selection/dependency
* __DNS Record TTL__ - Period of time, when the DNS record is cached in the browser, High TTL ~ 24h, Low TTL ~ 60s; It's about balancing DNS workload vs. outdated DNS records
* CNAME can be only non-root domain, that means at least 3rd level domain (2nd level domain is not working)
* Alias:
  * can pointed only within AWS resources (*.amazonaws.com)
  * Free of charge, Native Heath Checks

### Routing policies

* __Simple Routing Policy__: Single entry, or multiple entries. But all entries are return to client and client choose -> Client-side load balancing
* __Weighted Routing Policy__: Trafic distributed based on given weight; e.g. for A/B testing
__Latency Routing Policy__: To target with lowest latency; Latency is measured from user to AWS region
* __Health Checks__: Endpoint/Other Health Checks (calculated)/CloudWatch Alarm; 30s or 10s (more pricee)
* __Failover Routing Policy__: Mandatory exactly one Primary&Secondary entry; Health Check must be attached to control failover
* __Geolocation Routing Policy__: Based on the IP origin - country/continent
* __Geoproximity Routing Policy__: "BIAS" to add gravity to given region; shifting trafic from one region to another
* __Multivalue Routing Policy__: Up to 8 records; Mandatory Health Check association; All healthy targets are returned to client for client-side load balancing

### Registrar

* Route 53 is also domain registrator;
* But DNS is different from Registrar, even if offered within same service
* You can use also Route53 with another Registrar; you need to enter AWS nameservers into your domain entry

## Solution Architecture

### Well Architecture Framework

* Operational Excelence, Performance, Security, Reliability, Costs

### Speed-up EC2 instancines start

* __Golden AMI__ - all prepared app image
* __Bootstrap with User Data__: slower
* __Hybrid__: Golden AMI + User Data script to load some dynamic variables (URL, config, etc.)
* RDS/EBS Volume snapspots

### ElasticBeanStalk

* free, pay only for underlying instancies
* It's Contious Delivery Service
* Automated deployment and configuration


## S3

* Simple Storage Service
* Bucket = Directory
* Object = File
* Address: s3://bucket/object
* object consists of Full Key = Prefix + file key
* There is no directory hierarchy, only looks like it is
* S3 is region level (But console is global)
* Max 5TB object size
* Max 5GB upload size -> Multi-upload; recomended for > 100MB
* __Requestor Pays__ = Setting when downloader payes for corresponding networking costs (must be signed in AWS)

### S3 Versioning

* Defined on Bucket level
* NULL value for non versioned version of the file
* `Deletion Marker` instead of deletion; Deletion of `Deletion Marker` is pernament deletion

### S3 Encryption

* __SSE-S3__ Managed by S3
  * Server Side encryption
  * Set by header during upload
* __SSE-KMS__ Managed by KMS
  * Server Side encryption
  * Set by header during upload
* __SSE-C__ Custom managed keys
  * Server Side encryption
  * Set by header during upload
  * Key is always passed in header of all https requests
  * HTTPS is mandatory because requests contains encryption key
* __Client Side Entryption__
  * Fully managed by the client (with support of e.g. S3 Enctryption Library)
  * Not visible on AWS side, as AWS is not aware at all about such a encryption

* Default specified on bucket level
* Each obejct can have specific settings

### S3 Security

* __User Based__: Through AIM settings
* __Resource Based__
  * __S3 Bucket Policies__: bucket wide-rules from S3 Console, allows cross-account
    * Used to grant public access; force encryption; grant cross-account access 
  * __Object ACL__: fine grain
  * __Bucket ACL__: less common
* Access granted if allowed by IAM or by S3 policy; if ther is explicit DENY in IAM, there is not access
* MFA can be required for deletion 
* __Pre-Signed URLs__
  * URL valid only for limited time (e.g. download premium video for signed user)
  * Can be generated in CLI, SDK
  * Default 1 hour
  * Auth. are inherited from granting user
  * `aws s3 presign <object> <region> <expoiration>`
* Strong (ACID) consistency for file upload, listing, access, etc.
* Only Bucket owner can enable/disable option `MFA-Delete`, this can done only via CLI
* __S3 Lock__ & __Glacier Vault Lock__
  * Adopts WORM = Write Once Ready Many
  * The lock and the corresponding policy cannot be removed - good for compliance reasons
  * For S3 lock you need to: (1) Enable versioning (2) define __Retention Policy__ or set __Legal Hold__ (no expiration)
  * __Governance Mode__ = special auth. required to change settings
  * __Compliance Mode__ = not possible to change including root aws user (!)

### S3 Logging

* Never set your logging bucket (log storage) as monitored bucket (relevant for logging) -> it's infinite loop and creates huge data-volume
* By enable logging the log-storage buckget get automatically updated with write auth. for log develivery group
* Logs can be analyzed by Athena

### S3 Replicattion

* __CRR__ = Cross Region Replication
  * Compliance
  * Low latency
* __SRR__ = Same Region Replication
  * Log aggregration
  * Copy data to test/dev
* Must enable versioning, both source & target; Replication includes same object ID
* Can be between different accounts
* IAM write role to target bucket need to be provided
* Only new objects (after enablement) are replicated
* Deletion is not replicated; `Delete Marker` can be replicated
* There is no replication chainig - object can be replicated only once (WHY?)

### S3 Classess

* All classess can be used within one bucket
* S3 Standard
* S3 Standard-IA (lower garanteed avaiablity), retrival fee per GB
* S3 One Zone IA (lower avaiability and reability), retrival fee per GB
* S3 Intelligent Tiering - monthly re-sync between Standard and IA, retrival fee per GB
* S3 Glacier = Archive; Retrivals: Expedited <5 minutes; Standard < 5 hours; Bulk < 12 hours; Minumum storage 90 days, retrival fee per GB
* S3 Deep Archive - Retrivals Standard < 12 hours, Bulk < 48 hours; minimum storage 180 days, retrival fee per GB

### S3 Lifecycle Rules

* Objects can be moved between storage classes (e.g. move to IA after 60 days, move to Glacier after 180 days, etc.0)
* Can be also used to delete files, including individual versions OR clean-up multipart upload
* Moving is different from restoring!
* Can be assign based on prefix or tag
* Rules can target current or previous verions of object
* __S3 Analyics__ to improve your livecycle rules

### S3 Performance

* Very hight, granted per prefix
* If SSE-KMS used, there is performance limitation inherited from KMS (number of requests per second)
* Use AWS Edge Location (compatible with multi-upload)

## S3 Select & GLacier Select

* Limit data by server-side filtering (e.g. filter CSV file)
* Only simple queries = filter rows & columns
* Save network transfer, client side CPU

## S3 Event notification

* need to enable versioning to work properly
* You can sent notification to SNS, SQS, Lamdba
* Add access policy allowing S3 bucket to write to notified service

### Static websites

* Enabled by Console settings
* If part of embedded website - must be enabled by specif CORS policy (JSON)
* S3 BYte-Range Fetches = multiple Get request per byte range

## Athena

* Serveless analytics against S3 files
* Pricing per query and scanned data

### AWS CLI

* Default region: `us-east-1`


## Other Resources

* <https://quizlet.com/464209586/memory-mnemonics-flash-cards/>
* <https://memorang.com/flashcards/92826/AWS+Solutions+Architect>