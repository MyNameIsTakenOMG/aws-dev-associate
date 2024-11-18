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

- 

### practice tests

- practice exam:
  - 1st try: 61% (40/65)

### review lectures and tests







