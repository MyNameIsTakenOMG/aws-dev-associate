# aws-dev-associate

## road map
- [table of lecture contents](#table-of-lecture-contents)
- [practice tests](#pratice-tests)
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
#### route53
#### vpc
#### s3
#### aws cli sdk and iam roles and policies
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


### practice tests

- practice exam:
  - 1st try: 61% (40/65)

### review lectures and tests







