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

## Other Resources

* <https://quizlet.com/464209586/memory-mnemonics-flash-cards/>
* <https://memorang.com/flashcards/92826/AWS+Solutions+Architect>