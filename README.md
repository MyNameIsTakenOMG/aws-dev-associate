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

- moving between storage classes
  - using `lifecycle rules`
- lifecycle rules:
  - transition actions
  - expiration actions
  - can be create for certain prefix or object tags
- s3 analytics -- storage class analysis
  - recommendations for standard and standard IA
  - good first step to put together lifecycle rules
- s3 event notifications
  - sns
  - sqs
  - lambda
  - eventbridge:
    - advanced filtering options with json rules
    - multi-destinations
    - eventbridge capabilities
- s3 performance
  - 3,500 put/copy/post/delete per prefix
  - 5,500 get/head per prefix
  - multi-part upload: must use > 5gb
  - s3 transfer acceleration
  - s3 byte-range fetches
    - parallelize get requests
    - can be configured to only fetch part of the data
- s3 select & glacier select
  - less network transfer
  - can filter rows and columns
  - can retrieve less data using sql
- s3 user-defined object metadata & s3 object tags
  - we cannot search the object metadata or object tags
  - instead, we must use external db as a search index, such as dynamodb


#### s3 security

- object encryption
  - sse-s3: use the key managed and owned by aws s3
    - header: "x-amz-server-side-encryption": "AES256"
  - sse-kms: use aws kms to manage the key
    - header: "x-amz-server-side-encryption": "aws:kms"
    - audit using cloudtrail
    - kms limit
  - sse-c: encryption key is managed by you
    - https is a must
    - the key must be provided every request
    - aws only handle encryption
  - client-side encryption: cse
    - use client libraries like: Amazon S3 Client-Side Encryption Library
    - clients handle encryption and decryption
- encryption in transit
  - ssl/tls
  - bucket policy condition: `aws:SecureTransport:`
- default encryption vs bucket policies
  - **note**: bucket policies are evaluated before default encryption
- s3 cors
  - we need to enable correct cors headers for clients to fetch
- s3 MFA delete
  - only root account can enable/disable MFA delete
  - no need: enable versioning; list deleted versions
  - need: permanently delete version; suspend versioning
- s3 access logs
  - set a target bucket(different) for storing access logs
- pre-signed urls
  - use cli, sdk, console to create pre-signed urls
- s3 access points
  - has its own domain
  - an access point policy
- s3 access points -- vpc origin
  - between the vpc gateway endpoint or interface endpoint and s3 buckets
  - vpc endpoint policy must allow access to the target bucket and access point
- s3 object lambda
  - architecture:
    - client
    - s3 object lambda access point
    - lambda
    - s3 access point
  - use cases:
    - redacting personal info before send them back to the caller
    - converting data formats or resizing images

  
#### cloudfront

- overview:
  - cdn
  - DDoS protection, integrated with aws shield, aws waf
- origins
  - s3 buckets:
    - OAC
  - custom origins:
    - s3 websites
    - alb
    - ec2
    - any http backend
- cloudfront vs s3 cross-region replication
  - cloudfront: good for static content availablt anywhere
  - s3: good for dynamic content available in a few regions
- caching
  - at edge locations
  - identify each object using `cache key`
  - increase the cache hit ratio and decrease requests to the origin
  - invalidation: `CreateInvalidation` api
- cache key:
  - unique identifiers for each object
  - hostname + resource portion of the url
  - can add other elements: http headers, cookies, query strings using `cache policies`
- cache policies:
  - http headers:
    - none
    - whitelist
  - cookies
  - query strings
    - none
    - whitelist
    - inlude all-except
    - all (not good, too many)
  - control TTL, can set by the origin using `Cache-Control` header
  - custom policies support
  - **note**: all http headers, cookies, query strings in `cache key` are included in origin requests
- origin request policy
  - Specify values that you want to include in origin requests without
including them in the Cache Key (no duplicated cached content)
  - can include: http headers, cookies, query strings
  - ability to add cloudfront http headers and custom headers to an origin requst that was not included in the viewer request
  - support custom policy
- cache invalidations
  - we can force an entire or partial cache refresh by performing cloudfront invalidations
  - can specify invalidation path: `*`, `/images/*`
- cache behaviors
  - configure different settings based on the content type or path pattern: `/api/*`, `/*`
  - the default cache behavior is always the last to be process and is always `/*`
- geo restriction
  - restrict who can access your distribution:
    - allowlist
    - blocklist
- signed url / signed cookies (normally after authentication and authorization)
  - for example, only premium users can access paid shared content
  - can attach a policy with:
    - url expiration
    - ip ranges
    - trusted signers (which aws accounts can create signed url)
  - signed url: individual files
  - signed cookies: multi files
- cloudfront signed url vs s3 pre-signed url
  - cloudfront:
    - filter by ip, path, data, expire
    - define access path
    - only root user can manage it
    - utilize caching features
  - s3:
    - uses iam key
    - issue a request as the person who pre-signed the url
- cloudfront signed url process
  - signers:
    - a trusted key group(recommended)
    - an aws account that has a cloudfront key pair(not recommended, should not use root account)
  - can create multi key groups
  - generate key pair
- price classes
  - all
  - 200: most regions, exluding the most expensive ones
  - 100: only the least expensive ones
- multiple origins
  - route to different origins based on the content type
  - based on path pattern: `/images/*`, `/api/*`
- origin groups
  - increate HA and do failover
  - one origin group: one primary and one secondary origin
- field level encryption
  - protect sensitive user info for each request
  - encryption at the edge locations
  - specify fields(max 10) to encrypt and public key used for encryption
  - decryption at the web server side
- real-time logs
  - using KDS
  - choose:
    - sampling rate
    - specify fields and cache behaviors(path patterns)



#### containers

- overview:
  - apps are packaged in containers that can run on any os
  - docker image repositories:
    - docker hub
    - aws ecr
- docker vs virtual machines
  - docker containers are sharing the resources
- docker containers management on aws
  - aws ecs
  - aws eks
  - aws fargate: serverless container platform, working with ecs and eks
  - ecr: storing container images
- aws ecs
  - ec2 launch type
    - each ec2 must have ecs agent to register in the ecs cluster
    - aws takes care of starting / stopping containers
    - ecs tasks: launch ec2 instances on ecs clusters
  - fargate launch type
    - no ec2 instances to manage
    - just take care of task definitions
    - aws runs ecs tasks for your based on cpu / ram you need
    - to scale, just increase the number of tasks
  - iam roles
    - ec2 instance profile: (ec2 launch type only)
      - used by ecs agent
      - make api calls to ecs service
      - send logs to cloudwatch logs
      - pull images from ecr
      - reference data in secrets manager or ssm parameter store
    - ecs task role
      - each task has a role
      - defined in task definition
  - load balancer integrations
    - alb: most use cases
    - nlb: only for high thoughput
    - clb: not recommended
  - data volumes (efs: serverless)
    - mount EFS onto ecs tasks
    - works for ec2 and fargate types
    - **note**: s3 cannot be used as file system
  - ecs auto scaling
    - uses aws application auto scaling
    - based on : cpu, ram, or alb requests count per target
    - target tracking
    - step scaling
    - scheduling schaling
    - ecs auto scaling (task level)
    - ec2 auto scaling (instance level)
  - ec2 launch type -- auto scaling ec2 instances
    - scaling by adding ec2 instances
    - asg
    - ecs cluster capacity provider: paired with asg
  - rolling updates
    - min: minimum healthy percent (relative to the actual running capacity: 100%) 
    - max: maximum percent (actual running capacity + allowed to create tasks: extra)
  - integrated with eventbridge:
    - on-schedule
    - on-demand
  - integrated with sqs
  - task definitions:
    - json form to tell ecs how to run a docker container
    - cpu, ram, iam role, image, port, logging, env, networking
    - up to 10 containers in one task definition
  - load balancing:
    - ec2 launch type:
      - dynamic host port mapping if the container port is defined in the task definition
      - alb will find the right port
      - must allow ec2 security group to any port from alb security group
    - fargate
      - each task has a private ip
      - only container port
      - one iam role per task definition
  - environment variables
    - hardcoded
    - ssm parameter store
    - secrets manager
    - env files(bulk) -- s3
  - data volumes (bind mounts)
    - share data between multi container in the same task definition
    - works for both ec2 and fargate
  - task placement (only for ec2 launch type)
    - task placement contraints
      - distinctInstance: tasks are placed on different instances
      - memberOf: placed on ec2 instances with a specified expression; or uses the cluster query language(advanced)
    - task placement strategies: best effort
      - binpack: aim the least available amount of cpu and ram
      - random
      - spread: tasks are spread evenly based on the specified value: az
      - or mix them up
    - task placement process:
      - identify the proper ec2 instance
      - identify the instance with task placement contraints
      - identify the instance with task placement strategies
      - select the instance
- ecr
  - container images registry
  - integrated with ecs
  - public and private
- aws copilot
  - help run apps on AppRunner, ECS, and fargate
  - help focus on building apps rather than setting up infra
  - provision all required infra for containerized apps
  - auto-deploy using codepipeline
  - deploy to multiple env
  - troubleshooting, logs, health status
- aws eks
  - aws managed kubernates cluster
  - eks supports ec2 and fargate
  - cloud-agnostic
  - collect logs and metrics using cloudwatch container insights
  - Node Types:
    - managed node group
    - self-managed nodes: you create nodes and register to the cluster and then managed by asg
    - aws fargatea
  - data volumes
    - need to specify storageClass manifest on your eks cluster
    - leverages a container storage interface compliant driver
    - support:
      - ebs
      - efs (works with fargate)
      - fsx for lustre
      - fsx for netapp ontap



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







