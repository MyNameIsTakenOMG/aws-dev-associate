# aws-dev-associate

## road map
- [table of lecture contents](#table-of-lecture-contents)
- [lecture hands on](#lecture-hands-on)
- [practice tests](#practice-tests)
- [review lectures and tests](#review-lectures-and-tests)

### table of lecture contents
- [aws iam](#aws-iam)
- [ec2](#ec2)
- [ec2 instance storage](#ec2-instance-storage)
- [HA and scalability](#ha-and-scalability)
- [rds and aurora and elasticache](#rds-and-aurora-and-elasticache)
- [route53](#route53)
- [vpc](#vpc)
- [s3](#s3)
- [aws cli sdk and iam roles and policies](#aws-cli-sdk-and-iam-roles-and-policies)
- [s3 advanced](#s3-advanced)
- [s3 security](#s3-security)
- [cloudfront](#cloudfront)
- [containers](#containers)
- [elastic beanstalk](#elastic-beanstalk)
- [cloudformation](#cloudformation)
- [aws integration and messaging](#aws-integration-and-messaging)
- [aws monitoring and troubleshooting and audit](#aws-monitoring-and-troubleshooting-and-audit)
- [lambda](#lambda)
- [dynamodb](#dynamodb)
- [api gateway](#api-gateway)
- [cicd](#cicd)
- [serverless application model](#serverless-application-model)
- [cdk](#cdk)
- [cognito](#cognito)
- [other serverless](#other-serverless)
- [advanced identity](#advanced-identity)
- [security and encryption](#security-and-encryption)
- [other services](#other-services)


#### aws iam

- iam users & groups
  - user can belong to multiple groups or no group
  - groups can not be nested
- **Global service**
- iam permissions:
  - users & groups can be assigned `policies`
  - in aws, apply the least privilege principle
  - users who belong to multiple groups can inherit permissions from multiple groups policies
- iam password policy
  - allow all iam users to change their own passwords
  - prevent password re-use
  - more...
- mfa: password + code
  - mfa software(authy, google authenticator, support mutliple tokens)
  - hardware(universal 2nd factor:U2F security key, support multi-users with a single key)
  - hardware key fob mfa device: gemalto
  - hardware key fob mfa device for aws govcloud(US): surepassID
- access aws
  - aws console: password + mfa
  - cli: access key
  - sdk: access key
- iam roles for services
- iam security tools
  - iam credentials report: account level
  - iam access advisor: user level



#### ec2

- ec2 capabilities:
  - renting ec2 virtual machines
  - storing data: EBS
  - distributing load: ELB
  - scaling services: ASG
- ec2 sizing & configuration
  - windows, macos, linux
  - cpu, ram
  - storage: instance-store, ebs & efs
  - network card: public ip address
  - security group: firewall
  - user data
- ec2 user data: bootstrap script
  - only run once at the instance first start
  - runs with the root user permissions
- instance types
  - general purpose
  - compute optimized: HPC, ML, gaming server, batching workloads
  - memory optimized: databases, in-memory databases, big unstructured data real-time processing
  - storage optimized: OLTP, databases, in-memory databases, data warehouse, distributed file systems
- security groups
  - only contain `allow` rules
  - stateful, incoming traffic --> outgoing traffic
  - can be attached to multi-instances
  - application timeout error--> security group
  - all inbound traffic is blocked by default
- classic ports:
  - 22: ssh, sftp
  - 21: ftp
  - 80: http
  - 443: https
  - 3389: rdp (remote desktop procotol -- login windows instance)
- ssh & ec2 instance connect
- ec2 purchaing options
  - on-demand
  - reserved instances
  - saving plans
  - spot instances
  - dedicated hosts
  - dedicated instance
  - capacity reservations (combined with RI, Saving Plans)


#### ec2 instance storage

- ebs volume
  - network drive
  - az-scoped
  - have a provisioned capacity
  - root volume: delete on termination(by default)
  - snapshots: no need to detach, but recommended
    - snapshot features:
      - archive
      - recycle bin
      - fast snapshot restore
- ami
  - customization of ec2 instances
  - region-scoped
- instance store:
  - better I/O
  - good for buffer, cache
- ebs volume types
  - gp2/gp3
  - io1/io2: support ebs multi-attach (up to 16 ec2)
  - st1: big data, data warehouse,...
  - sc1: infrequently access
  - gp and io can be used as root volumes
- efs:
  - multi-az
  - nfs protocol
  - for linux
  - encryption using kms
- efs performance & storage classes
  - performance mode
  - throughput mode
  - storage classes: storage tiers, availability


#### HA and scalability

- scalability: vertical, horizontal
- availability: goes hand in hand with horizontal scaling, (running system in at least 2 data centers)
- ELB
  - health check: done on a port and a endpoint
  - types:
    - CLB
    - ALB:
      - layer 7;
      - http/2 and websocket;
      - route to different target groups: ec2(or ec2 in asg), ecs, lambda, private ip
      - fixed hostname
      - to see the client ip, use header: `X-Forwarded-For`
    - NLB: layer 4 (udp, tcp); handle millions of requests per requests; has `one static ip per az`; support EIP(helpful for whitelisting ip addresses)
      - target groups: ec2, private ip, ALB,
      - health check: tcp, http, https
    - GWLB: layer 3; use `GENEVE` procotol on port `6081`
      - manage a fleet of 3rd party network virtual appliance in aws
      - firewalls, intrusion detection, deep packet inspection system
      - target groups: ec2, private ip
  - security group
  - sticky sessions: works for clb, alb, nlb
    - application-based cookies: created by targets(custom cookies) or load balancers(application cookies)
    - duration-based cookies: created by load balancers
  - cross-zone load balancing
    - alb: enabled by default
    - nlb: disabled by default, no free
    - clb: disabled
  - ssl/tls
    - can manage certs using acm
    - or upload your own certs
    - https listener:
      - must specify a default cert
      - can optionally add a list of certs for multi domains
      - client can use SNI(server name indication) to specify which hostname to reach
      - alb and nlb: support multi https listeners; use sni
      - clb: support only one ssl, 
  - ssl -- SNI
    - only works for alb and nlb, cloudfront
  - connection draining(deregistration delay): time to complete `in-flight requests` while the instance is de-registering or unhealthy 
- ASG
  - a launch template
  - cloudwatch alarms & scaling
  - scaling policies:
    - dynamic scaling:
      - target tracking
      - simple/step
    - scheduled scaling
    - predictive scaling
  - metrics for scaling: `CPUUtilization`, `RequestCountPerTarget`, `Average Network In / Out`, or custom metrics
  - scaling cooldowns: when a scaling activity happends, you are in the cooldown period(no launch or terminate instances)
    - use Golden AMIs
  - instance refresh:
    - update launch template and re-create all ec2 instances
    - using native feature: `instance refresh`
    - setting minimum healthy percentage
    - specify `warm-up` time
 

#### rds and aurora and elasticache

- RDS: relational db service
  - support multi different db engines, including aws aurora
  - managed service
  - cannot ssh to the underlying instances
  - storage auto scaling:
    - have to set `Maximum Storage Threshold`
  - read replicas:
    - up to 15
    - within az, cross az or cross region
    - replication is `async`
    - apps must update db connection string
    - replicas can be promoted to their own db
    - use case: data analytics
    - network cost: same-region, free; cross-region charged
  - multi-az (Disaster Recovery):
    - replication: `sync`
    - one DNS name
    - not used for scaling
    - just a standby
    - from single-az to multi-az: click `modify`, then a snapshot is created, then a new db is created, then a synchronization is established
  - rds proxy:
    - fully managed db proxy
    - pool and share db connections
    - reduce rds & aurora failover time
    - must be access from vpc (private)
- Aurora:
  - support mysql and postgres
  - HA & read scaling:
    - 6 copies across 3 az:
      - 4/6 needed for write
      - 3/6 needed for read
      - up to 15 read replicas
      - auto failover
      - cross-region replication
  - aurora db cluster:
    - writer endpoint
    - reader endpoint
  - rds & aurora security:
    - at-rest encryption
    - in-flight encryption
    - iam authentication
    - security groups
    - no ssh except on `rds custom`
    - audit logs can be enabled and sent to cloudwatch logs
- elasticache:
  - managed redis:
    - multi-az and auto failover
    - read replicas with HA
    - backup and restore
  - managed memcached:
    - mutli-node
    - no HA
    - no persistant
    - no backup and restore
    - multi-threading
  - cache must have an invalidation strategy to make sure only the most current data is used in there
  - cache implementation considerations:
    - outdated data
    - data changing slowly or rapidly
    - data structured well or not
  - lazy loading/ cache-aside/ lazy population:
    - cons: cache read miss cause 3 round trips; data could be stale 
  - write through -- add or update cache when db is updated:
    - cons: data missed until it is added or updated in the main db (using lazy loading to solve it); a lot of data will never be read since each time db is updated, it will also update the cache
  - cache evictions and TTL
    - occur in three ways:
      - delete explicitly
      - memory is full and it is not recently used
      - a TTL is set
    - use cases: leaderboards; comments; activity stream
    - consider to scale up or out if eviction happens too frequently
  - final words of wisdom:
    - lazy loading works for many cases, especially on the read side
    - write-through is usually combined with lazy loading as targeted for the queries or workloads
    - set a ttl is usually not a bad idea, except when using write-through
    - only cache things that make sense

  
#### route53

- dns terminologies:
  - domain registar
  - dns records
  - zone file: contains dns records
  - name server: resovle dns queries
  - TLD
  - SLD
- route53:
  - 100 SLA
  - health check
  - records:
    - domain/subdomain name
    - record type
    - value
    - routing policy
    - ttl
  - records types
    - A
    - AAAA
    - Alias
    - CNAME
    - NS
  - hosted zones
    - public
    - private
    - $0.5 per month per hosted zone
  - TTL:
    - except for Alias records
    - high ttl or low ttl
  - CNAME vs Alias
    - CNAME: not for TLD
    - Alias: TLD or non-TLD, point to aws resources
  - Alias records:
    - map hostnames to aws resources
    - no ttl
    - TLD or non-TLD
    - targets:
      - **Note**: not for ec2 instances
      - elb
      - cloudfront
      - api gateway
      - EB
      - s3 websites
      - vpc interface endpoint
      - global accelerator
      - route53 records in the same hosted zone
  - health check:
    - http health check only for public resources
    - but can integrate with cloudwatch metrics to monitor private resources
    - automated dns failover
      - monitor an endpoint:
        - around 15 global health checkers
        - can be configured to only check on the first 5120 bytes of the responses
      - calculated health checks (similar to composite alarms)
      - monitor cloudwatch alarms
  - traffic flow:
    - visual editor to manage complex routing decision trees
    - can be saved as traffic flow policy and applied to different route 53 hosted zones
  - routing policies
    - simple: no health check; route traffic to a single resource
    - weighted:
      - dns records must have the same name and type
      - health check
      - all zero, then traffic evenly distributed
      - if zero, then no traffic sent
      - weights no need to be 100 when summed up
    - latency-based:
      - health check (failover feature)
      - helpful when latency is a priority
    - failover(active-passive)
    - geolocation:
      - should have a default record
      - health check
    - geoproximity
      - shift traffic to resources based on **bias**
      - must use **route 53 traffic flow**
    - ip-based routing
      - routing is based on ip addresses
      - provide a list of CIDRs for clients
    - multi-value:
      - health check
      - same record name and type
- route 53 and other dns registar
  - register a domain name and then use route 53 name servers



#### vpc

- overview -- basics:
  - vpc: private network
  - public subnet
  - private subnet
  - route tables
- internet gateway
- nat gateway (or nat instances:self-managed)
- NACL & security group
  - NACL: allow or deny; subnet level; stateless
  - security group: only allow; ec2 instance/ENI; stateful
- vpc flow logs:
  - vpc flow logs, subnet flow logs, ENI flow logs
  - logs can go to s3, KDF, cloudwatch logs
- vpc peering:
  - connect two vpc, privately using aws network
  - must not have overlapping CIDR
  - not transitive
- vpc endpoints:
  - gateway endpoints: s3, dynamodb
  - interface endpoint: for DX, s2s vpn
    - public virtual interface
    - private virtual interface
- s2s vpn & DX
  - s2s vpn: on-prem to aws; use public internet; encrypted
  - DX: take a long time to build; private connection; physical connection; private network



#### s3

- buckets:
  - regional service
  - needs a global unique name
- objects
  - the key is the full path: prefix + object name
  - max object size: 5tb
  - muse use `multi-part upload` if size > 5gb
- s3 security:
  - user-based: iam policies
  - resources-based:
    - bucket policies
    - object access control list -- fine-grained
    - bucket access control list -- less common
  - encryption
- s3 static website hosting:
  - make sure the bucket policy allows public access
  - make sure the bucket name is the same as the record name
- s3 versioning
  - enable at bucket level
- s3 -- replication
  - must enable versioning
  - cross-region replication
  - same-region replication
  - asynchronous copy
  - using `s3 batch replication` to replicate existing objects
  - can replicate `delete marker`
  - there is no chaining replicaiton
- s3 storage classes
  - standard
  - infrequent access:
    - standard-IA
    - one zone-IA
  - glacier storage classes
    - instant retrieval
    - flexible retrieval
    - deep archive
  - intelligent-tiering


#### aws cli sdk and iam roles and policies

- ec2 instance metadata (IMDS)
  - allow ec2 to `learn about themselves` without using an iam role
  - can retrieve the iam role from the metadata, but not the iam policy
- IMDS v2 and IMDS v1
- MFA with CLI
  - must create a temporary session using `sts getsessiontoken` api call
- aws sdk
- aws limits (quotas)
  - api rate limits
  - service quotas (service limits)
- exponential backoff
  - use exponential backoff if you encounter `ThrottlingException`
  - retry logic has been included in aws sdk (not for conditional check)
    - must only implement the retry for 5xx errors and throttling
    - do not implement on 4xx client errors
- aws cli credentials provider chain
  - command line options
  - environment variables
  - cli credentials file -- aws configure: `~/.aws/credentials`, `~/.aws/config`
  - container credentials for ecs tasks
  - instance profile credentials for ec2 instance profiles
- aws sdk default credentials provider chain(java)
  - java system properties
  - environment variables: access_key, access_secret
  - default credential profile file: for example `~/.aws/credentials`
  - ecs container credentials for ecs containers
  - instance profile credentials for ec2 instances
- aws credentials best practices
  - never store aws credentials in your code
  - if working with aws, use iam roles
  - if working outside of aws, use environment variables/ named profiles
- sign aws api requests
  - if use sdk or cli, the requests are signed for us



#### s3 advanced
#### s3 security
#### cloudfront
#### containers
#### elastic beanstalk
#### cloudformation
#### aws integration and messaging
#### aws monitoring and troubleshooting and audit
#### lambda
#### dynamodb
#### api gateway
#### cicd
#### serverless application model
#### cdk
#### cognito
#### other serverless
#### advanced identity
#### security and encryption
#### other services


### lecture hands on


### practice tests

- practice exam:
  - 1st try: 61% (40/65)

### review lectures and tests







