# SOA-C03 Study Guide: AWS Certified CloudOps Engineer – Associate

A free, open study guide for the **AWS Certified CloudOps Engineer – Associate (SOA-C03)** exam: revision notes for every domain, side-by-side comparisons of commonly confused services, a glossary, 20 worked sample questions, and the official syllabus as a checklist with a full free lesson for every topic.

Maintained by [SaveMyCert](https://www.savemycert.com/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide), where you can read every lesson free, [practice with explained questions](https://www.savemycert.com/practice/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) and [take timed mock exams](https://www.savemycert.com/mocks/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [Exam domains](#exam-domains)
- [What is in this repo](#what-is-in-this-repo)
- [Syllabus checklist](#syllabus-checklist)
  - [Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization](#domain-1-monitoring-logging-analysis-remediation-and-performance-optimization)
  - [Domain 2: Reliability and Business Continuity](#domain-2-reliability-and-business-continuity)
  - [Domain 3: Deployment, Provisioning, and Automation](#domain-3-deployment-provisioning-and-automation)
  - [Domain 4: Security and Compliance](#domain-4-security-and-compliance)
  - [Domain 5: Networking and Content Delivery](#domain-5-networking-and-content-delivery)
- [How to study for SOA-C03](#how-to-study-for-soa-c03)
- [Sample questions](sample-questions.md)
- [Free resources](#free-resources)

## Exam at a glance

| | |
|---|---|
| Exam code | SOA-C03 |
| Level | Associate |
| Questions | 65 |
| Time limit | 130 min |
| Passing score | 720 / 1000 |
| Format | Multiple choice & response |
| Exam fee | $150 |
| Valid for | 3 years |

Exam details change. Always confirm them in the official [AWS Certified CloudOps Engineer – Associate (SOA-C03) exam guide](https://docs.aws.amazon.com/aws-certification/latest/sysops-administrator-associate-03/sysops-administrator-associate-03.html) from Amazon Web Services.

## Exam domains

| # | Domain | Weight | Topics |
|---|---|---|---|
| 1 | [Monitoring, Logging, Analysis, Remediation, and Performance Optimization](#domain-1-monitoring-logging-analysis-remediation-and-performance-optimization) | 22% | 3 |
| 2 | [Reliability and Business Continuity](#domain-2-reliability-and-business-continuity) | 22% | 3 |
| 3 | [Deployment, Provisioning, and Automation](#domain-3-deployment-provisioning-and-automation) | 22% | 2 |
| 4 | [Security and Compliance](#domain-4-security-and-compliance) | 16% | 2 |
| 5 | [Networking and Content Delivery](#domain-5-networking-and-content-delivery) | 18% | 3 |

That is 5 domains and 13 topics. Spend your time in proportion to the weights: the heaviest domain decides more of your score than the lightest.

## What is in this repo

| File | What it gives you |
|---|---|
| [Domain 1 notes](notes/domain-1-monitoring-logging-analysis-remediation-and-performance-optimization.md) | Monitoring, Logging, Analysis, Remediation, and Performance Optimization: condensed revision notes per topic |
| [Domain 2 notes](notes/domain-2-reliability-and-business-continuity.md) | Reliability and Business Continuity: condensed revision notes per topic |
| [Domain 3 notes](notes/domain-3-deployment-provisioning-and-automation.md) | Deployment, Provisioning, and Automation: condensed revision notes per topic |
| [Domain 4 notes](notes/domain-4-security-and-compliance.md) | Security and Compliance: condensed revision notes per topic |
| [Domain 5 notes](notes/domain-5-networking-and-content-delivery.md) | Networking and Content Delivery: condensed revision notes per topic |
| [Commonly confused services](comparisons.md) | Side-by-side tables of the services questions set against each other |
| [Glossary](glossary.md) | Every in-scope term and service in one sentence |
| [Sample questions](sample-questions.md) | 20 worked questions with answers and reasoning |
| [Exam-day guide](exam-day-guide.md) | Booking, testing options, scoring, results and retakes |

## Syllabus checklist

Tick each topic off once you can explain it without notes. The "Must know" facts are the ones questions turn on. Each lesson link goes to the complete, free lesson.

### Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization

**Weight: 22%.** CloudWatch-centered observability — metrics, alarms, and logs — plus automated remediation and performance tuning for compute, storage, and databases.

📝 Revision notes: [Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization](notes/domain-1-monitoring-logging-analysis-remediation-and-performance-optimization.md)

- [ ] **1.1 Implement metrics, alarms, and filters by using AWS monitoring and logging services**
  <br>Monitoring and logging for serverless, compute, and AI workloads with CloudWatch, CloudTrail, and Amazon Managed Service for Prometheus; configuring and managing the CloudWatch agent for EC2, ECS, and EKS metrics/logs; CloudWatch alarms (including composite alarms) that invoke services directly or via EventBridge, with SNS notifications; shareable cross-account, cross-Region dashboards.
  - 📖 Lesson: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Default EC2 metrics never include memory or disk-space utilization — collecting those (and process counts via procstat) requires the CloudWatch agent.
  - Must know: Run the agent at fleet scale with one Parameter Store configuration (AmazonCloudWatch-*), AWS-ConfigureAWSPackage to install, and AmazonCloudWatch-ManageAgent to apply; instances need CloudWatchAgentServerPolicy.
- [ ] **1.2 Identify and remediate issues by using monitoring and availability metrics**
  <br>Analyzing performance metrics and automating remediation with CloudWatch, Lambda, Systems Manager, CloudTrail, Kiro, and AWS DevOps Agent; routing, enriching, and delivering events with EventBridge and troubleshooting event bus rules; creating and running custom and predefined Systems Manager Automation runbooks.
  - 📖 Lesson: [Automated Remediation: EventBridge, Runbooks, and EC2 Recovery (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-automated-remediation-eventbridge-runbooks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: The recover action attaches only to a StatusCheckFailed_System alarm: it keeps instance ID, IP addresses and EBS volumes, loses RAM, instance store data and uptime, and is not supported for Auto Scaling group members or Dedicated Hosts.
  - Must know: StatusCheckFailed_Instance means an OS-level problem — a system-check alarm never fires for it; use the reboot action, and repair an unbootable OS offline with AWSSupport-ExecuteEC2Rescue or a helper instance.
- [ ] **1.3 Implement performance optimization strategies for compute, storage, and database resources**
  <br>Optimizing compute via performance metrics, resource tags, and AWS tools (EC2 instance selection, placement groups); analyzing and optimizing EBS performance and volume types; S3 performance strategies (DataSync, Transfer Acceleration, multipart uploads, lifecycle policies); selecting and optimizing shared storage (EFS, FSx, Amazon S3 Files, EFS lifecycle policies); monitoring RDS with Performance Insights and CloudWatch alarms and tuning with proactive recommendations and RDS Proxy.
  - 📖 Lesson: [EC2, EBS, S3 and RDS Performance Optimization for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-ec2-ebs-rds-performance-optimization/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Trusted Advisor only flags idle or low-utilization instances; Compute Optimizer turns CloudWatch history into specific EC2, EBS and Lambda recommendations; use a 32-day lookback (free) for monthly peaks, and publish memory with the CloudWatch agent or it is ignored.
  - Must know: Tags make findings and spend attributable, but a tag reaches Cost Explorer only after it is activated as a cost allocation tag; grouping starts at activation unless you request a backfill (up to 12 months).

### Domain 2: Reliability and Business Continuity

**Weight: 22%.** Scalability and elasticity, highly available and resilient environments, and backup/restore strategies that meet RTO and RPO.

📝 Revision notes: [Domain 2: Reliability and Business Continuity](notes/domain-2-reliability-and-business-continuity.md)

- [ ] **2.1 Implement scalability and elasticity**
  <br>Configuring and managing scaling mechanisms across compute environments; caching for dynamic scalability (CloudFront, ElastiCache); scaling in managed databases (RDS, DynamoDB).
  - 📖 Lesson: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: "Keep a metric near a value" → target tracking, on a metric that falls as capacity is added: average CPU, ALBRequestCountPerTarget (not total RequestCount), or SQS backlog per instance.
  - Must know: Backlog per instance target = acceptable wait ÷ processing time per message, a number of messages (120 s ÷ 4 s = 30).
- [ ] **2.2 Implement highly available and resilient environments**
  <br>Configuring and troubleshooting Elastic Load Balancing and Route 53 health checks; configuring fault-tolerant systems such as Multi-AZ deployments.
  - 📖 Lesson: [High Availability: ELB and Route 53 Health Checks, Multi-AZ Design](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-high-availability-health-checks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: UDP, a static IP per AZ or PrivateLink → NLB; path, host or header routing → ALB; transparent appliance insertion → Gateway Load Balancer. An NLB can front an ALB through an ALB-type target group.
  - Must know: ALB listener rules run from the lowest priority number, first match wins; one HTTPS listener serves many domains with SNI certificates and host-header rules.
- [ ] **2.3 Implement backup and restore strategies**
  <br>Automating snapshots and backups for EC2, RDS, EBS, S3, and DynamoDB with services like AWS Backup; restoring databases (e.g. point-in-time restore) to meet RTO, RPO, and cost requirements; versioning for storage services (S3, FSx); disaster-recovery procedures and best practices (backup and restore, pilot light, warm standby, active/active).
  - 📖 Lesson: [Backup, Restore, and Disaster Recovery Strategies on AWS](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-backup-restore-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: RPO = maximum acceptable data loss, RTO = maximum acceptable downtime; worst-case RPO equals the backup interval, and copying a recovery point never makes it fresher.
  - Must know: AWS Backup centralizes schedules (hourly at most frequent), retention, copies and lifecycle; tag-based assignment and organization backup policies (with the IAM role pre-created in every account) are the at-scale answers.

### Domain 3: Deployment, Provisioning, and Automation

**Weight: 22%.** Provisioning with CloudFormation and the AWS CDK, image building, cross-account resource sharing, and event-driven operational automation.

📝 Revision notes: [Domain 3: Deployment, Provisioning, and Automation](notes/domain-3-deployment-provisioning-and-automation.md)

- [ ] **3.1 Provision and maintain cloud resources**
  <br>Creating and managing AMIs and container images (EC2 Image Builder); managing resource stacks with CloudFormation and the AWS CDK and remediating deployment issues (subnet sizing, CloudFormation errors, permissions); provisioning and sharing resources across Regions and accounts (AWS RAM, CloudFormation StackSets); deployment strategies and third-party automation tools (Terraform, Git).
  - 📖 Lesson: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: AMIs are Regional: copy them ahead of need; a copy is re-encrypted with a destination-Region key and must be shared again.
  - Must know: Share encrypted AMIs only under a customer managed key that target accounts can use — never aws/ebs; share with an OU ARN to cover future accounts.
- [ ] **3.2 Automate the management of existing resources**
  <br>Automating operational processes with Systems Manager; event-driven automation with Lambda, S3 Event Notifications, EventBridge, and AWS DevOps Agent.
  - 📖 Lesson: [Systems Manager and event-driven automation: managing resources for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-systems-manager-event-driven-automation/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: A managed node needs a running (recent enough) SSM Agent, credentials from an instance profile or Default Host Management Configuration (IMDSv2, agent 3.2.582.0+), and a path to the ssm, ssmmessages and ec2messages endpoints.
  - Must know: Interface endpoints only work with private DNS enabled and a security group allowing HTTPS 443 from the nodes; S3, KMS and Logs endpoints serve patching, encryption and logging, not registration.

### Domain 4: Security and Compliance

**Weight: 16%.** IAM policy management and access auditing, multi-account security strategies, and protecting data and infrastructure.

📝 Revision notes: [Domain 4: Security and Compliance](notes/domain-4-security-and-compliance.md)

- [ ] **4.1 Implement and manage security and compliance tools and policies**
  <br>IAM features — password policies, MFA, roles, federated identity, resource policies, policy conditions; troubleshooting and auditing access with CloudTrail, IAM Access Analyzer, and the IAM policy simulator; secure multi-account strategies (AWS Organizations, service control policies, IAM Identity Center); remediating Trusted Advisor security checks; enforcing compliance requirements and continuous monitoring (Region and service selections, AWS Config conformance packs).
  - 📖 Lesson: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Only identity and resource policies grant access — SCPs, RCPs, permissions boundaries and session policies can only cap it, and an explicit deny anywhere always wins.
  - Must know: Enforce MFA with a Deny using BoolIfExists on aws:MultiFactorAuthPresent; plain Bool or MultiFactorAuthAge fail open for access-key requests, and CLI users get MFA-backed credentials from sts get-session-token.
- [ ] **4.2 Implement strategies to protect data and infrastructure**
  <br>Data classification schemes; encryption at rest (AWS KMS) and in transit (ACM); securely storing secrets with AWS services; configuring reports and remediating findings across Security Hub, GuardDuty, AWS Config, Inspector, and AWS Security Agent.
  - 📖 Lesson: [Data Protection on AWS: KMS, ACM, Secrets, and Security Services (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-data-protection-security-services/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Classification is a tag: tag policies standardize it, tag-scoped (organization) Config rules enforce and report on it, and Macie checks that S3 data matches its label.
  - Must know: Macie automated discovery samples daily for broad coverage; a discovery job with 100% sampling depth inspects every object in the buckets you choose. Customer managed keys must let the Macie role decrypt.

### Domain 5: Networking and Content Delivery

**Weight: 18%.** VPC networking and private connectivity, DNS and content delivery, and troubleshooting with flow logs and network monitoring.

📝 Revision notes: [Domain 5: Networking and Content Delivery](notes/domain-5-networking-and-content-delivery.md)

- [ ] **5.1 Implement and optimize networking features and connectivity**
  <br>Configuring a VPC — subnets, route tables, network ACLs, security groups, NAT gateways, internet gateway, egress-only internet gateway; private networking connectivity (VPC endpoints, AWS PrivateLink, VPC peering); auditing network protection services (Route 53 Resolver DNS Firewall, WAF, Shield, Network Firewall) in a single account; optimizing network architecture cost.
  - 📖 Lesson: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: A subnet is public only if its route table sends 0.0.0.0/0 to an internet gateway and its instances have public IPs.
  - Must know: Every subnet loses five IPs; subnet and primary VPC CIDRs are fixed, so add a secondary VPC CIDR and new subnets when addresses run out.
- [ ] **5.2 Configure domains, DNS services, and content delivery**
  <br>Configuring DNS with Route 53 Resolver; Route 53 routing policies, configurations, and query logging; content and service distribution with CloudFront and Global Accelerator.
  - 📖 Lesson: [Route 53 DNS and CloudFront Content Delivery for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-route53-dns-cloudfront-delivery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Alias records work at the zone apex, track their AWS target automatically, and are free to query when targeting AWS resources; CNAMEs cannot sit at the apex.
  - Must know: Lower a record's TTL at least one old-TTL period before a cutover; lowering it at the moment of change leaves resolvers on the old answer.
- [ ] **5.3 Troubleshoot network connectivity issues**
  <br>Troubleshooting VPC configurations (subnets, route tables, network ACLs, security groups, transit gateways, NAT gateways); interpreting networking logs (VPC flow logs, ELB access logs, WAF web ACL logs, CloudFront logs, container logs); remediating CloudFront caching issues; troubleshooting hybrid and private connectivity; configuring and analyzing CloudWatch network monitoring.
  - 📖 Lesson: [AWS Network Troubleshooting: VPC Flow Logs and the Layered Diagnosis Method](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-network-troubleshooting-vpc-flow-logs/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Security groups are stateful; network ACLs are stateless — a rejected reply to an accepted request is always a NACL problem.
  - Must know: Network ACLs never filter Route 53 Resolver (VPC+2), instance metadata, DHCP or Amazon Time Sync traffic.

## How to study for SOA-C03

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [SOA-C03 revision notes](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide).
2. **Practice straight after reading.** Answer [SOA-C03 practice questions](https://www.savemycert.com/practice/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [SOA-C03 mock exam](https://www.savemycert.com/mocks/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [SOA-C03 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked SOA-C03 questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [AWS Certified CloudOps Engineer – Associate (SOA-C03) exam guide](https://docs.aws.amazon.com/aws-certification/latest/sysops-administrator-associate-03/sysops-administrator-associate-03.html): the official source (Amazon Web Services)
- [SOA-C03 certification overview](https://www.savemycert.com/certifications/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
- [SOA-C03 revision notes](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide): every lesson, free to read
- [SOA-C03 practice questions](https://www.savemycert.com/practice/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide): with an explanation on every option
- [SOA-C03 mock exams](https://www.savemycert.com/mocks/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide): full-length and timed
- [SOA-C03 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide): the key facts on one page
- [All certification study guides](https://github.com/savemycert/certification-study-guides)

## Contributing

Spotted an error or an out-of-date fact? [Open an issue](../../issues) with the topic and a link to the official source. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and disclaimer

This guide is licensed under [CC BY 4.0](LICENSE). You can reuse and adapt it, including commercially, as long as you credit **SaveMyCert** with a link to https://www.savemycert.com/.

This is an independent study resource. It is not affiliated with or endorsed by Amazon Web Services. AWS Certified CloudOps Engineer – Associate and SOA-C03 are trademarks of their respective owner. Exam domains and weights are taken from the official exam guide linked above.
