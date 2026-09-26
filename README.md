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

Exam details change. Always confirm them in the official [AWS Certified CloudOps Engineer – Associate (SOA-C03) certification page](https://aws.amazon.com/certification/certified-cloudops-engineer-associate/) from Amazon Web Services.

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
  <br>Monitoring and logging with CloudWatch, CloudTrail, and Amazon Managed Service for Prometheus; configuring and managing the CloudWatch agent for EC2, ECS, and EKS metrics/logs; CloudWatch alarms (including composite alarms) that invoke services directly or via EventBridge, with SNS notifications; shareable cross-account, cross-Region dashboards.
  - 📖 Lesson: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Default EC2 metrics never include memory or disk-space utilization — collecting those requires the CloudWatch agent.
  - Must know: Standard EC2 monitoring is 5-minute metrics; detailed monitoring is 1-minute, costs extra, and adds frequency, not new metrics.
- [ ] **1.2 Identify and remediate issues by using monitoring and availability metrics**
  <br>Analyzing performance metrics and automating remediation with CloudWatch, AWS User Notifications, Lambda, Systems Manager, CloudTrail, and auto scaling; routing, enriching, and delivering events with EventBridge and troubleshooting event bus rules; creating and running custom and predefined Systems Manager Automation runbooks.
  - 📖 Lesson: [Automated Remediation: EventBridge, Runbooks, and EC2 Recovery (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-automated-remediation-eventbridge-runbooks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: StatusCheckFailed_System means AWS infrastructure under the instance failed — the recover action migrates it to healthy hardware, preserving instance ID, IP addresses, and EBS volumes.
  - Must know: StatusCheckFailed_Instance means an OS-level problem (exhausted memory, corrupted filesystem, bad network config, failed kernel) — recovery can't fix it; reboot or repair the OS.
- [ ] **1.3 Implement performance optimization strategies for compute, storage, and database resources**
  <br>Optimizing compute via performance metrics, resource tags, and AWS tools (EC2 instance selection, placement groups); analyzing and optimizing EBS performance and volume types; S3 performance strategies (DataSync, Transfer Acceleration, multipart uploads, lifecycle policies); selecting and optimizing shared storage (EFS, FSx, EFS lifecycle policies); monitoring RDS with Performance Insights and CloudWatch alarms and tuning with proactive recommendations and RDS Proxy.
  - 📖 Lesson: [EC2, EBS, S3 and RDS Performance Optimization for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-ec2-ebs-rds-performance-optimization/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Compute Optimizer turns CloudWatch history into over/under-provisioned classifications with specific EC2, EBS, and Lambda recommendations — Trusted Advisor only flags, it doesn't size.
  - Must know: A T-family instance that slows under sustained load has drained its CPU credits — check CPUCreditBalance; fix with unlimited mode or a fixed-performance family.

### Domain 2: Reliability and Business Continuity

**Weight: 22%.** Scalability and elasticity, highly available and resilient environments, and backup/restore strategies that meet RTO and RPO.

📝 Revision notes: [Domain 2: Reliability and Business Continuity](notes/domain-2-reliability-and-business-continuity.md)

- [ ] **2.1 Implement scalability and elasticity**
  <br>Configuring and managing scaling mechanisms across compute environments; caching for dynamic scalability (CloudFront, ElastiCache); scaling in managed databases (RDS, DynamoDB).
  - 📖 Lesson: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: "Keep a metric near a value" (for example, average CPU at 50%) → target tracking, the default scaling-policy answer.
  - Must know: Simple scaling is legacy: one adjustment per cooldown; prefer target tracking or step scaling for reactive load.
- [ ] **2.2 Implement highly available and resilient environments**
  <br>Configuring and troubleshooting Elastic Load Balancing and Route 53 health checks; configuring fault-tolerant systems such as Multi-AZ deployments.
  - 📖 Lesson: [High Availability: ELB and Route 53 Health Checks, Multi-AZ Design](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-high-availability-health-checks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: "Static IP per AZ", UDP, or extreme TCP performance → NLB; path, host, or header routing → ALB.
  - Must know: All targets unhealthy is systemic: health-check path or port, a security group blocking probes from the ALB, or an app-wide non-2xx response.
- [ ] **2.3 Implement backup and restore strategies**
  <br>Automating snapshots and backups for EC2, RDS, EBS, S3, and DynamoDB with services like AWS Backup; restoring databases (e.g. point-in-time restore) to meet RTO, RPO, and cost requirements; versioning for storage services (S3, FSx); disaster-recovery procedures.
  - 📖 Lesson: [Backup, Restore, and Disaster Recovery Strategies on AWS](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-backup-restore-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: RPO = maximum acceptable data loss, RTO = maximum acceptable downtime — translate every scenario into these two numbers first.
  - Must know: AWS Backup centralizes schedules, retention, cold-storage lifecycle, and cross-Region/cross-account copies; tag-based resource assignment is the at-scale answer.

### Domain 3: Deployment, Provisioning, and Automation

**Weight: 22%.** Provisioning with CloudFormation and the AWS CDK, image building, cross-account resource sharing, and event-driven operational automation.

📝 Revision notes: [Domain 3: Deployment, Provisioning, and Automation](notes/domain-3-deployment-provisioning-and-automation.md)

- [ ] **3.1 Provision and maintain cloud resources**
  <br>Creating and managing AMIs and container images (EC2 Image Builder); managing resource stacks with CloudFormation and the AWS CDK and remediating deployment issues (subnet sizing, CloudFormation errors, permissions); provisioning and sharing resources across Regions and accounts (AWS RAM, CloudFormation StackSets); deployment strategies and third-party automation tools (Terraform, Git).
  - 📖 Lesson: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Deregistering an AMI does not delete its EBS snapshots — delete them separately or storage costs continue.
  - Must know: A change set previews every Add/Modify/Remove and flags Replacement — a replaced resource is deleted and recreated, so check stateful resources first.
- [ ] **3.2 Automate the management of existing resources**
  <br>Automating operational processes with Systems Manager; event-driven automation with Lambda and S3 Event Notifications.
  - 📖 Lesson: [Systems Manager and event-driven automation: managing resources for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-systems-manager-event-driven-automation/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: A managed instance needs three things: a running SSM Agent, an instance profile with AmazonSSMManagedInstanceCore, and a network path to the Systems Manager endpoints.
  - Must know: Shell access without SSH keys, bastion hosts, or open inbound ports → Session Manager, with sessions logged to CloudWatch Logs or S3.

### Domain 4: Security and Compliance

**Weight: 16%.** IAM policy management and access auditing, multi-account security strategies, and protecting data and infrastructure.

📝 Revision notes: [Domain 4: Security and Compliance](notes/domain-4-security-and-compliance.md)

- [ ] **4.1 Implement and manage security and compliance tools and policies**
  <br>IAM features — password policies, MFA, roles, federated identity, resource policies, policy conditions; troubleshooting and auditing access with CloudTrail, IAM Access Analyzer, and the IAM policy simulator; secure multi-account strategies; remediating Trusted Advisor security checks; enforcing compliance requirements such as Region and service restrictions.
  - 📖 Lesson: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Only IAM identity and resource policies grant access — SCPs and permissions boundaries can only cap it, and an explicit deny anywhere always wins.
  - Must know: Enforce MFA with a Deny policy conditioned on aws:MultiFactorAuthPresent — the account password policy cannot require MFA.
- [ ] **4.2 Implement strategies to protect data and infrastructure**
  <br>Data classification schemes; encryption at rest (AWS KMS) and in transit (ACM); securely storing secrets with AWS services; configuring reports and remediating findings across Security Hub, GuardDuty, AWS Config, and Inspector.
  - 📖 Lesson: [Data Protection on AWS: KMS, ACM, Secrets, and Security Services (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-data-protection-security-services/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: You cannot encrypt an existing EBS volume or RDS instance in place — snapshot, copy the snapshot with encryption, restore, and cut over.
  - Must know: The KMS key policy is the primary authority: an IAM allow is useless if the key policy neither permits the principal nor delegates to IAM.

### Domain 5: Networking and Content Delivery

**Weight: 18%.** VPC networking and private connectivity, DNS and content delivery, and troubleshooting with flow logs and network monitoring.

📝 Revision notes: [Domain 5: Networking and Content Delivery](notes/domain-5-networking-and-content-delivery.md)

- [ ] **5.1 Implement and optimize networking features and connectivity**
  <br>Configuring a VPC — subnets, route tables, network ACLs, security groups, NAT gateways, internet gateway, egress-only internet gateway; private networking connectivity; auditing network protection services (Route 53 Resolver DNS Firewall, WAF, Shield, Network Firewall) in a single account; optimizing network architecture cost.
  - 📖 Lesson: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: A subnet is public only if its route table sends 0.0.0.0/0 to an internet gateway and its instances have public IPs.
  - Must know: Every subnet loses five IPs: network address, VPC router, DNS resolver, future-use, and broadcast.
- [ ] **5.2 Configure domains, DNS services, and content delivery**
  <br>Configuring DNS with Route 53 Resolver; Route 53 routing policies, configurations, and query logging; content and service distribution with CloudFront and Global Accelerator.
  - 📖 Lesson: [Route 53 DNS and CloudFront Content Delivery for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-route53-dns-cloudfront-delivery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Alias records work at the zone apex and are free to query when targeting AWS resources; CNAMEs cannot sit at the apex.
  - Must know: On-prem must resolve VPC names: inbound Resolver endpoint. VPCs must resolve on-prem domains: outbound endpoint plus forwarding rules.
- [ ] **5.3 Troubleshoot network connectivity issues**
  <br>Troubleshooting VPC configurations (subnets, route tables, network ACLs, security groups, transit gateways, NAT gateways); interpreting networking logs (VPC flow logs, ELB access logs, WAF web ACL logs, CloudFront logs, container logs); remediating CloudFront caching issues; troubleshooting hybrid and private connectivity; configuring and analyzing CloudWatch network monitoring.
  - 📖 Lesson: [AWS Network Troubleshooting: VPC Flow Logs and the Layered Diagnosis Method](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-network-troubleshooting-vpc-flow-logs/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)
  - Must know: Security groups are stateful; network ACLs are stateless — a rejected reply to an accepted request is always a NACL problem.
  - Must know: ACCEPT then REJECT for the same flow in VPC Flow Logs means a NACL is blocking the return on ephemeral ports 1024-65535.

## How to study for SOA-C03

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [SOA-C03 revision notes](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide).
2. **Practice straight after reading.** Answer [SOA-C03 practice questions](https://www.savemycert.com/practice/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [SOA-C03 mock exam](https://www.savemycert.com/mocks/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [SOA-C03 cheat sheet](https://www.savemycert.com/cheat-sheet/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked SOA-C03 questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [AWS Certified CloudOps Engineer – Associate (SOA-C03) certification page](https://aws.amazon.com/certification/certified-cloudops-engineer-associate/): the official source (Amazon Web Services)
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
