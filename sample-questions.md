# SOA-C03 Sample Questions with Answers

20 worked practice questions for the **AWS Certified CloudOps Engineer – Associate (SOA-C03)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [SOA-C03 practice questions](https://www.savemycert.com/practice/aws-cloudops-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide) on SaveMyCert.

## Question 1

*Monitoring, Logging, Analysis, Remediation, and Performance Optimization*

**An on-call engineer is investigating a report of high memory usage on an EC2 instance, but the CloudWatch console shows CPUUtilization and NetworkIn for the instance and no memory metrics at all. What should the engineer do to make memory utilization available in CloudWatch?** Choose one.

- **a.** Create a metric filter on the instance's log group.
- **b.** Attach an IAM role with broader CloudWatch permissions to the instance.
- **c.** Install and configure the CloudWatch agent on the instance.
- **d.** Enable detailed monitoring on the instance.

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Metric filters count pattern matches in log data that already exists. They cannot generate memory utilization measurements that nothing is collecting.
- **b** ❌ Permissions alone do not produce metrics. Without the agent installed, no process inside the OS is measuring or publishing memory usage.
- **c** ✅ Memory is an in-guest metric that the hypervisor cannot see. The CloudWatch agent runs inside the operating system and publishes memory, disk-space, and swap metrics to CloudWatch.
- **d** ❌ Detailed monitoring only raises the frequency of the existing hypervisor metrics from 5 minutes to 1 minute. It adds no new metrics, so memory still would not appear.

**The concept.** Default EC2 metrics come from the hypervisor, which can only observe the instance from outside: CPU, network, disk I/O, and status checks. Memory utilization, disk-space utilization, and swap usage are internal to the guest OS, so they are never in the default metric set.

**Why this is correct.** Installing and configuring the CloudWatch agent is the only way to collect in-guest metrics such as memory, because the agent runs inside the operating system and publishes what the hypervisor cannot see. Detailed monitoring is tempting but only increases the frequency of the same hypervisor metrics; a metric filter works on logs, not OS measurements; and IAM changes grant permission without creating any data source.

**How to reason it out**

1. Confirm the missing metric is an in-guest metric (memory, disk space, swap) rather than a hypervisor metric.
2. Install the CloudWatch agent on the instance.
3. Configure the agent's JSON configuration to collect the memory metrics required.
4. Verify the new metrics arrive in CloudWatch and build alarms or dashboards on them.

> **Exam tip:** Memory and disk-space metrics missing from CloudWatch always point to installing the CloudWatch agent — detailed monitoring never adds new metrics.

</details>

📖 Learn this topic: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 2

*Monitoring, Logging, Analysis, Remediation, and Performance Optimization*

**A CloudOps team must collect memory and disk-space metrics from 300 EC2 instances and keep the collection configuration identical across the entire fleet. Which approach meets the requirement with the LEAST operational overhead?** Choose one.

- **a.** Connect to each instance over SSH, install the agent, and copy a local configuration file.
- **b.** Enable detailed monitoring on all instances from the console.
- **c.** Deploy a custom script to each instance that calls PutMetricData with memory values.
- **d.** Store the CloudWatch agent configuration in Systems Manager Parameter Store and use Systems Manager to install and configure the agent across the fleet.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Hand-configuring 300 instances is slow, requires SSH access, and local config files drift apart over time — the opposite of a consistent fleet.
- **b** ❌ Detailed monitoring increases metric frequency to 1 minute but never adds memory or disk-space metrics, which only the agent can collect.
- **c** ❌ Custom scripts duplicate what the agent already does, and 300 per-instance copies of homegrown code invite drift and maintenance burden.
- **d** ✅ One configuration in Parameter Store gives every instance an identical setup, and Systems Manager installs and configures the agent at scale without logging in to any instance.

**The concept.** The CloudWatch agent reads a JSON configuration describing which metrics and log files to collect. The operational best practice for fleets is to store that configuration centrally in Systems Manager Parameter Store and to install and configure the agent through Systems Manager rather than by hand.

**Why this is correct.** Parameter Store gives the whole fleet one authoritative agent configuration, and Systems Manager performs the installation and configuration at scale — no SSH, no per-instance drift. SSHing to each instance is unmanageable at 300 instances; detailed monitoring cannot produce in-guest metrics at all; and a custom PutMetricData script reinvents the agent with code the team must now maintain.

**How to reason it out**

1. Author the CloudWatch agent JSON configuration listing the memory and disk metrics to collect.
2. Store the configuration as a parameter in Systems Manager Parameter Store.
3. Use Systems Manager to install the agent across the fleet.
4. Configure the agent on every instance to load the shared parameter, then verify metrics arrive.

> **Exam tip:** Fleet-wide agent rollout means Systems Manager for installation and Parameter Store for one shared agent configuration.

</details>

📖 Learn this topic: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 3

*Monitoring, Logging, Analysis, Remediation, and Performance Optimization*

**A single stateful EC2 instance hosting a licensing service begins failing its system status check, and AWS reports a problem with the physical host's hardware. The team wants the instance brought back automatically while keeping its private IP address. Which configuration achieves this?** Choose one.

- **a.** Place the instance in an Auto Scaling group with minimum and maximum size of 1.
- **b.** A CloudWatch alarm on the StatusCheckFailed_Instance metric with the reboot action.
- **c.** An alarm that notifies an SNS topic so the on-call can follow the recovery runbook.
- **d.** A CloudWatch alarm on the StatusCheckFailed_System metric with the EC2 recover action.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Auto Scaling replaces unhealthy instances with new ones, which get a new instance ID and private IP — wrong for a stateful instance that must keep its identity.
- **b** ❌ This watches the wrong check — the failure is at the infrastructure layer — and rebooting on the same failed host cannot cure a hardware problem.
- **c** ❌ Paging a human is notification, not automatic remediation — the requirement is that the instance comes back without anyone logging in.
- **d** ✅ The recover action responds to system-check failures by migrating the instance to healthy hardware while preserving its instance ID, private and Elastic IP addresses, and attached EBS volumes.

**The concept.** StatusCheckFailed_System indicates a problem with the AWS infrastructure under the instance — power, network, or hardware on the physical host. The matching remediation is the alarm's recover action, which migrates the instance to healthy hardware.

**Why this is correct.** An alarm on StatusCheckFailed_System with the recover action automatically moves the instance to new underlying hardware, keeping the instance ID, private and Elastic IPs, and EBS volumes — the licensing service returns with its identity intact. Watching the instance check with a reboot targets the wrong failure layer; Auto Scaling replacement destroys the stateful instance's identity; and SNS-only wiring leaves the fix manual.

**How to reason it out**

1. Confirm the failure is the system status check (infrastructure), not the instance check (OS).
2. Create a CloudWatch alarm on StatusCheckFailed_System for the instance.
3. Attach the EC2 recover action to the alarm, plus an SNS notification so the team is informed in parallel.
4. After a recovery, verify the alarm returns to OK and the service resumed on the preserved IP.

> **Exam tip:** Host-level hardware failure = StatusCheckFailed_System = the recover action, which preserves instance ID, IPs, and EBS volumes.

</details>

📖 Learn this topic: [Automated Remediation: EventBridge, Runbooks, and EC2 Recovery (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-automated-remediation-eventbridge-runbooks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 4

*Monitoring, Logging, Analysis, Remediation, and Performance Optimization*

**A CloudOps engineer is auditing a fleet of 300 EC2 instances after a cost review. Management wants specific alternative instance type recommendations for each instance, generated from the instances' CloudWatch utilization history, with the LEAST manual analysis. Which AWS service should the engineer use?** Choose one.

- **a.** AWS Config
- **b.** Amazon CloudWatch dashboards
- **c.** AWS Compute Optimizer
- **d.** AWS Trusted Advisor

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ AWS Config records resource configuration changes and evaluates compliance rules; it has no view of performance utilization and produces no sizing recommendations.
- **b** ❌ Dashboards visualize metrics but make no recommendations — the engineer would still have to analyze 300 instances by hand, which violates the least-manual-analysis constraint.
- **c** ✅ Compute Optimizer applies machine learning to CloudWatch utilization history, classifies each instance as over-provisioned, under-provisioned, or optimized, and recommends specific alternative instance types with projected utilization.
- **d** ❌ Trusted Advisor flags idle and low-utilization instances as cost checks, but it does not analyze utilization history to recommend specific replacement instance types.

**The concept.** AWS Compute Optimizer is the service purpose-built for right-sizing: it applies machine learning to CloudWatch utilization history and produces specific instance-type recommendations for EC2 (plus recommendations for EBS volumes, Lambda memory, and Auto Scaling groups), classifying each resource as over-provisioned, under-provisioned, or optimized.

**Why this is correct.** Compute Optimizer is correct because it is the only option that turns historical utilization data into concrete replacement instance types automatically, satisfying the least-manual-analysis requirement across 300 instances. Trusted Advisor is the classic trap: it only flags low-utilization and idle instances as cost checks — it identifies candidates but does not size replacements. CloudWatch dashboards show the raw metrics with no judgment attached, and AWS Config tracks configuration state and compliance, not performance.

**How to reason it out**

1. Opt in to AWS Compute Optimizer for the account (or organization).
2. Let it analyze the fleet's CloudWatch utilization history.
3. Review each instance's classification (over-provisioned, under-provisioned, optimized) and the recommended alternative instance types with projected utilization.
4. Schedule resizing changes for the over- and under-provisioned instances.

> **Exam tip:** When a question asks which tool recommends specific instance types from utilization history, the answer is Compute Optimizer — Trusted Advisor only flags, it does not size.

</details>

📖 Learn this topic: [EC2, EBS, S3 and RDS Performance Optimization for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-ec2-ebs-rds-performance-optimization/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 5

*Reliability and Business Continuity*

**A CloudOps engineer manages a web application on an EC2 Auto Scaling group. Management requires that the fleet's average CPU utilization stay near 50% as traffic rises and falls throughout the day. Which scaling configuration meets this requirement with the LEAST operational overhead?** Choose one.

- **a.** Step scaling policies with CloudWatch alarms at 60%, 70%, and 80% CPU
- **b.** A simple scaling policy that adds one instance when CPU exceeds 50%
- **c.** Scheduled scaling actions that add instances during business hours
- **d.** A target tracking scaling policy with a target of 50% average CPU utilization

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Step scaling requires the engineer to design and maintain the alarms and adjustment tiers, and it reacts to breaches rather than holding a metric near a value — more overhead for a worse fit.
- **b** ❌ Simple scaling is the legacy option: it makes one fixed adjustment and then waits out a cooldown before acting again, so it lags load changes and never manages capacity around a target.
- **c** ❌ Scheduled scaling changes capacity at fixed times regardless of actual CPU, so it cannot keep the average near 50% when real traffic deviates from the schedule.
- **d** ✅ Target tracking is built for exactly this: you set the target value and AWS creates and manages the CloudWatch alarms, scaling out and in to hold the metric near 50% with no further configuration.

**The concept.** Target tracking scaling holds a chosen metric — such as average CPU utilization — at a target value by automatically creating and managing the CloudWatch alarms and scaling in both directions.

**Why this is correct.** The requirement 'keep a metric near a value' maps directly to target tracking, which needs only the metric and the target number — the least configuration of any policy type. Step scaling could approximate the behavior but demands hand-built alarms and tiered adjustments; simple scaling is legacy and makes only one adjustment per cooldown window; scheduled scaling follows the clock, not the metric.

**How to reason it out**

1. Translate the requirement: 'keep average CPU near 50%' is a target-value goal, not a threshold or schedule.
2. Create a target tracking policy on the Auto Scaling group with ASGAverageCPUUtilization as the metric and 50 as the target.
3. Let AWS manage the generated alarms — do not edit or delete them.
4. Set instance warmup so booting instances do not distort the aggregate metric.

> **Exam tip:** When a requirement says keep a metric near a value, the answer is a target tracking policy — the modern default with the least operational overhead.

</details>

📖 Learn this topic: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 6

*Reliability and Business Continuity*

**An operations team wants an Auto Scaling group to add one instance when average CPU exceeds 70% but add four instances at once when average CPU exceeds 90%. Which scaling policy type supports this tiered response?** Choose one.

- **a.** Simple scaling
- **b.** Scheduled scaling
- **c.** Step scaling
- **d.** Target tracking scaling

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Simple scaling ties one alarm to one fixed adjustment and then waits out a cooldown, so it cannot respond differently to a small breach versus a large one.
- **b** ❌ Scheduled scaling changes capacity at specific times on a schedule; it does not react to CPU at all.
- **c** ✅ Step scaling defines step adjustments tied to alarm breach magnitude — +1 instance for the 70–90% band and +4 above 90% — so the response scales with severity.
- **d** ❌ Target tracking takes a single target value and manages its own alarms; it does not let you define different adjustment sizes for different breach severities.

**The concept.** Step scaling sizes the capacity adjustment to how far the metric breached the alarm threshold, using step adjustments you define.

**Why this is correct.** Only step scaling lets you say 'a small breach adds one instance, a large breach adds four' — the adjustment scales with breach magnitude. Target tracking holds one value and hides its alarms; simple scaling makes exactly one fixed adjustment per cooldown; scheduled scaling ignores metrics entirely.

**How to reason it out**

1. Identify the requirement: different response sizes at different severity levels of the same metric.
2. Create a CloudWatch alarm on average CPU for the Auto Scaling group.
3. Attach a step scaling policy with step adjustments: +1 instance for the 70–90% band, +4 instances above 90%.
4. Configure instance warmup so newly launched instances are counted correctly during multi-step responses.

> **Exam tip:** Tiered responses sized to breach severity are the signature use case for step scaling.

</details>

📖 Learn this topic: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 7

*Reliability and Business Continuity*

**A company runs a multiplayer game backend on EC2 instances that communicate with clients over UDP. Partner networks require a fixed IP address in each Availability Zone that they can add to their firewall allowlists. Which load balancer should a CloudOps engineer deploy?** Choose one.

- **a.** A Network Load Balancer
- **b.** An Application Load Balancer with an Elastic IP address attached to each node
- **c.** An Application Load Balancer
- **d.** One Application Load Balancer per Availability Zone behind Route 53 weighted records

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The NLB operates at layer 4 and supports UDP listeners, and it provides a static IP address per Availability Zone with Elastic IP support — exactly what fixed-IP firewall allowlists require.
- **b** ❌ ALBs do not support attaching Elastic IP addresses; only the NLB offers Elastic IP support and a static IP per AZ.
- **c** ❌ The ALB is a layer 7 load balancer for HTTP and HTTPS only — it cannot forward UDP traffic, and it exposes only a DNS name whose underlying IPs change.
- **d** ❌ Multiplying ALBs does not solve either problem: ALBs still cannot handle UDP, and each still presents changing IPs behind a DNS name.

**The concept.** The ALB-versus-NLB choice comes down to the layer where decisions are made. The NLB works at layer 4 (TCP, UDP, TLS), delivers extreme connection performance, and provides a static IP per Availability Zone with Elastic IP support. The ALB works at layer 7 (HTTP/HTTPS) with content-based routing but offers only a DNS name.

**Why this is correct.** This scenario contains two hard NLB signals: UDP traffic and a fixed IP per AZ for allowlisting. Only the NLB satisfies both. The ALB fails on protocol alone — it handles only HTTP and HTTPS. Attaching Elastic IPs to an ALB is not a supported feature, so that option is a fabricated capability. Spreading ALBs across AZs behind Route 53 changes nothing fundamental: each ALB is still HTTP-only and still resolves to changing IP addresses.

**How to reason it out**

1. Identify the protocol: UDP immediately rules out the Application Load Balancer.
2. Identify the addressing requirement: a fixed IP per AZ for firewall allowlists means static IPs, which only the NLB provides.
3. Create a Network Load Balancer with a UDP listener and enable the AZs the fleet uses.
4. Optionally assign an Elastic IP per AZ so the allowlisted addresses are ones the company controls permanently.
5. Share the per-AZ IP addresses with the partner networks for their firewall rules.

> **Exam tip:** UDP, non-HTTP protocols, or a static IP per AZ in the requirements → Network Load Balancer.

</details>

📖 Learn this topic: [High Availability: ELB and Route 53 Health Checks, Multi-AZ Design](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-high-availability-health-checks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 8

*Reliability and Business Continuity*

**After a post-incident review, company leadership mandates that the order-processing workload must experience no more than 30 minutes of data loss and must be back online within 2 hours of any failure. A CloudOps engineer is recording these targets in the disaster recovery plan. How should the engineer document them?** Choose one.

- **a.** RTO of 30 minutes and RPO of 2 hours
- **b.** RPO of 30 minutes and RTO of 2 hours
- **c.** Two RPO targets: a 30-minute RPO for transactions and a 2-hour RPO for the service
- **d.** Two RTO targets: a 30-minute RTO for data and a 2-hour RTO for the application

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ This reverses the two objectives. The 30-minute figure describes data loss, which maps to RPO, and the 2-hour figure describes downtime, which maps to RTO.
- **b** ✅ Maximum acceptable data loss expressed as time is the recovery point objective (RPO), and maximum acceptable downtime is the recovery time objective (RTO). "No more than 30 minutes of data loss" is the RPO; "back online within 2 hours" is the RTO.
- **c** ❌ RPO measures acceptable data loss only. The 2-hour limit on how long the service may be down is a downtime target, which is the RTO.
- **d** ❌ RTO measures downtime only. A limit on how much data may be lost is a separate objective — the RPO — not a second flavor of RTO.

**The concept.** RTO is the maximum acceptable time between failure and full restoration of service; RPO is the maximum acceptable data loss measured as time — how old the most recent recoverable copy may be.

**Why this is correct.** The 30-minute data-loss limit maps to RPO and the 2-hour recovery deadline maps to RTO, so option A is correct. Option B reverses the mapping, which is the classic trap. Options C and D collapse both requirements into a single objective type, but downtime and data loss are independent dimensions — a fast restore of an old backup meets RTO while blowing RPO, and vice versa.

**How to reason it out**

1. Read the scenario and find the sentence describing data loss — that number is the RPO.
2. Find the sentence describing how quickly service must return — that number is the RTO.
3. Record both objectives separately, because backup frequency drives RPO while restore speed drives RTO.
4. Use the two numbers to eliminate any backup or DR design that cannot meet either one.

> **Exam tip:** "How much data can we lose" is RPO; "how long can we be down" is RTO — translate every scenario into these two numbers first.

</details>

📖 Learn this topic: [Backup, Restore, and Disaster Recovery Strategies on AWS](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-backup-restore-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 9

*Deployment, Provisioning, and Automation*

**A CloudOps engineer deregistered dozens of outdated AMIs during an account cleanup, but the next monthly bill still shows the same EBS snapshot storage charges. What is the cause?** Choose one.

- **a.** Deregistering an AMI does not delete its backing EBS snapshots, so the engineer must delete the snapshots separately.
- **b.** EBS snapshot charges always continue until the source volumes are also deleted.
- **c.** The AMIs were shared with other accounts, which forces AWS to keep the snapshots billed to the owner.
- **d.** Deregistered AMIs are retained for 30 days before AWS automatically removes their snapshots.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ AMI deregistration only removes the ability to launch from the image; the underlying EBS snapshots remain and continue to accrue storage charges until they are deleted explicitly.
- **b** ❌ Snapshots are billed independently of their source volumes; deleting or keeping the volume has no effect on snapshot storage charges.
- **c** ❌ Sharing controls launch permissions, not snapshot lifecycle; the charges continue simply because the snapshots were never deleted, shared or not.
- **d** ❌ There is no automatic 30-day snapshot cleanup tied to AMI deregistration; the snapshots persist indefinitely until someone deletes them.

**The concept.** Deregistering an AMI and deleting its backing EBS snapshots are two separate operations, so stale-image cleanup is always a two-step job.

**Why this is correct.** Deregistration only prevents new launches from the image; the EBS snapshots that back it stay in the account and keep billing until deleted. The 30-day retention answer invents a lifecycle that does not exist, the sharing answer confuses launch permissions with storage lifecycle, and the source-volume answer gets snapshot billing backwards — snapshots are independent of volumes.

**How to reason it out**

1. Identify the snapshot IDs associated with each deregistered AMI (they are listed in the AMI's block device mappings before deregistration, or findable by description).
2. Deregister the AMI so no new instances can launch from it.
3. Delete the now-orphaned EBS snapshots explicitly.
4. Confirm in billing or Cost Explorer that snapshot storage charges drop.

> **Exam tip:** Deregistering an AMI never deletes its EBS snapshots — delete them separately or keep paying for them.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 10

*Deployment, Provisioning, and Automation*

**A company's disaster recovery plan requires the ability to launch its hardened web server image in a second Region. The AMI currently exists only in us-east-1. What should a CloudOps engineer do?** Choose one.

- **a.** Copy the AMI from us-east-1 to the DR Region so it exists locally there.
- **b.** Share the AMI with the DR Region from the AMI's permissions settings.
- **c.** Reference the existing AMI ID in launch templates created in the DR Region.
- **d.** Store the AMI in an S3 bucket with cross-Region replication enabled.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ AMIs are Regional resources, so cross-Region use requires an explicit AMI copy into each target Region, producing a new local AMI ID to reference in DR launch templates.
- **b** ❌ AMI sharing grants launch permissions to other AWS accounts, not to other Regions; there is no share-with-Region operation.
- **c** ❌ AMI IDs are Region-specific; a us-east-1 AMI ID is meaningless in any other Region and the launch would fail.
- **d** ❌ AMIs are backed by EBS snapshots managed by EC2, not by S3 objects you can replicate; this is not how AMIs move between Regions.

**The concept.** AMIs are Regional resources: to launch the same golden image in another Region you must copy the AMI there, which creates a new AMI with a new Region-local ID.

**Why this is correct.** Copying the AMI to the DR Region is the only mechanism that makes the image launchable there. Referencing the original ID fails because AMI IDs are Region-scoped, S3 replication is unrelated to how EC2 stores images, and sharing is an account-level permission concept — there is no such thing as sharing an AMI with a Region.

**How to reason it out**

1. Initiate an AMI copy from us-east-1 to the DR Region.
2. Wait for the copy to complete; note the new AMI ID minted in the DR Region.
3. Update DR launch templates or CloudFormation mappings to reference the new Region-local AMI ID.
4. Repeat the copy after each golden-image rebuild, or automate distribution with EC2 Image Builder.

> **Exam tip:** Cross-Region DR with AMIs means copying the AMI into each Region — every copy gets its own local AMI ID.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 11

*Deployment, Provisioning, and Automation*

**A security mandate prohibits SSH access to production EC2 instances: no key pairs, no bastion hosts, and no open inbound administration ports. Engineers still need an interactive shell, and every session must be auditable. Which solution meets the requirements?** Choose one.

- **a.** Deploy a bastion host in a public subnet with its security group restricted to office IP ranges.
- **b.** Allow inbound port 22 only from the corporate VPN CIDR range.
- **c.** Use Session Manager for shell access, with session logging enabled to CloudWatch Logs or S3.
- **d.** Use EC2 Instance Connect to push short-lived SSH keys to the instances.

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A bastion host is explicitly prohibited, and it still relies on SSH keys and an open inbound port — restricting the source IPs does not change that.
- **b** ❌ This is still SSH over an open inbound port with distributed keys; narrowing the source range does not satisfy any of the three prohibitions.
- **c** ✅ Session Manager brokers sessions over the SSM Agent's outbound connection — no keys, bastions, or inbound ports — with IAM controlling access and session logging providing the audit trail.
- **d** ❌ Instance Connect is still SSH: it pushes temporary keys and requires inbound SSH connectivity to the instance, violating the no-SSH, no-inbound-port mandate.

**The concept.** Session Manager provides interactive shell access without SSH keys, bastion hosts, or any open inbound ports, because the SSM Agent maintains an outbound connection that sessions are brokered over; IAM governs access and sessions can be logged to CloudWatch Logs or S3.

**Why this is correct.** Only Session Manager satisfies all four requirements at once: no keys, no bastion, no inbound ports, and a full audit trail of who connected and what they ran. The bastion answer violates the mandate outright, Instance Connect is still SSH with inbound port 22, and a VPN-scoped security group rule merely narrows an inbound path the mandate says must not exist.

**How to reason it out**

1. Confirm the instances are managed by Systems Manager (agent, instance profile, network path).
2. Grant engineers IAM permissions to start sessions, scoping by tags or resource where needed.
3. Enable session logging to CloudWatch Logs or an S3 bucket in Session Manager preferences.
4. Block all inbound administrative ports in the security groups — sessions ride the agent's outbound connection.

> **Exam tip:** No SSH, no bastion, no inbound ports, full audit → Session Manager with session logging.

</details>

📖 Learn this topic: [Systems Manager and event-driven automation: managing resources for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-systems-manager-event-driven-automation/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 12

*Deployment, Provisioning, and Automation*

**A security team has approved a hardened base AMI, and two other AWS accounts in the company need to launch instances from it. The image must not become publicly available. What is the correct way to grant access?** Choose one.

- **a.** Add the two account IDs to the AMI's launch permissions.
- **b.** Create a resource share for the AMI in AWS RAM.
- **c.** Make the AMI public and tell only the two teams the AMI ID.
- **d.** Copy the AMI's snapshots into an S3 bucket and grant the accounts bucket access.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ AMIs are shared by granting launch permissions to specific account IDs (or an organization), which lets those accounts launch from the image while it stays private to everyone else.
- **b** ❌ AWS RAM shares resources such as subnets and Transit Gateways; AMI sharing is done through the image's own launch permissions, not RAM.
- **c** ❌ A public AMI is launchable by any AWS account that discovers it, which directly violates the requirement to keep the image private.
- **d** ❌ AMIs are not distributed as S3 objects; granting bucket access would not make the image launchable in the other accounts.

**The concept.** Private AMI sharing works through launch permissions: the owner adds specific AWS account IDs (or shares across an organization), and those accounts can then launch instances from the image.

**Why this is correct.** Adding the account IDs to the AMI's launch permissions grants exactly the required access with no public exposure. AWS RAM is the tempting distractor because it is the multi-account sharing service — but AMIs are not shared through RAM; making the image public fails the privacy requirement outright; and S3 bucket access has nothing to do with how EC2 launches from an AMI.

**How to reason it out**

1. Open the AMI's permissions (or use the CLI modify-image-attribute operation).
2. Add the two target account IDs as launch permissions, keeping the image private.
3. If the AMI's snapshots are encrypted, ensure the KMS key policy also allows the target accounts.
4. Have the target accounts confirm they can see and launch the shared AMI.

> **Exam tip:** Share an AMI with specific accounts via launch permissions — AWS RAM does not share AMIs, and public sharing is all-or-nothing.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 13

*Security and Compliance*

**A security audit requires that every IAM user in an AWS account authenticate with multi-factor authentication (MFA) before performing any actions. An administrator opens the account password policy settings but cannot find any option to require MFA. How should the administrator enforce the requirement?** Choose one.

- **a.** Enable a Trusted Advisor security check that blocks sign-ins made without MFA.
- **b.** Attach an IAM policy to all users that denies every action except the MFA-management actions whenever the aws:MultiFactorAuthPresent condition key is not true.
- **c.** Enable the "Require MFA" option in the account password policy.
- **d.** Enable an AWS Config managed rule that checks whether IAM users have MFA enabled.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Trusted Advisor only flags findings, such as a root user without MFA; it never blocks any sign-in or API call.
- **b** ✅ This is the AWS-recommended enforcement pattern: a Deny using BoolIfExists on aws:MultiFactorAuthPresent locks users out of everything except registering an MFA device until they authenticate with MFA.
- **c** ❌ No such option exists. The account password policy governs only console password rules — length, complexity, reuse, and expiry — and has no MFA setting.
- **d** ❌ AWS Config is detective — it reports users without MFA as noncompliant but cannot force anyone to authenticate with MFA before acting.

**The concept.** The IAM account password policy cannot require MFA — it only controls console password rules. MFA is enforced with an IAM policy that denies actions when the request lacks MFA context.

**Why this is correct.** A Deny statement conditioned on aws:MultiFactorAuthPresent (using BoolIfExists so access-key requests, which never carry MFA context, are also denied) blocks every action except MFA registration until the user authenticates with MFA. The password policy has no MFA option, and AWS Config and Trusted Advisor are detective tools — they report problems but cannot prevent an unauthenticated action.

**How to reason it out**

1. Create an IAM policy with a Deny on all actions, excepting the MFA-management actions users need to register a device.
2. Add the condition BoolIfExists aws:MultiFactorAuthPresent is false, so both console and access-key requests without MFA are denied.
3. Attach the policy to all IAM users, typically through a group.
4. Verify a test user can only manage MFA until authenticating with a registered device.
5. Enable MFA on the root user separately — it is not covered by IAM user policies.

> **Exam tip:** The password policy cannot require MFA — enforce it with a Deny conditioned on aws:MultiFactorAuthPresent.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 14

*Security and Compliance*

**During a credential review, a CloudOps engineer finds that an application running on an EC2 instance reads an IAM user's access keys from a configuration file to call Amazon S3. What is the MOST secure way to eliminate the long-term credentials?** Choose one.

- **a.** Rotate the access keys every 90 days and keep them in the configuration file.
- **b.** Attach an IAM role to the instance through an instance profile and delete the IAM user's access keys.
- **c.** Move the access keys into the instance's user data script.
- **d.** Encrypt the configuration file with AWS KMS so the keys are not stored in plaintext.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Rotation shortens the exposure window but the credentials remain long-term secrets sitting in an artifact — the risk the review flagged is still present.
- **b** ✅ An instance profile delivers short-lived, automatically rotated role credentials to the application, so there are no long-term keys left to leak or rotate.
- **c** ❌ User data is readable through the instance metadata service and the console, so this makes the exposure worse while keeping the long-term keys.
- **d** ❌ The application must still decrypt and use the same long-term keys at runtime, so nothing about the credential model changes — the keys still exist and can still leak.

**The concept.** Workloads should obtain AWS credentials from IAM roles — for EC2, via an instance profile — instead of long-term access keys stored in files, AMIs, or user data.

**Why this is correct.** An instance profile role gives the application short-lived credentials that AWS rotates automatically, removing the long-term keys entirely. Rotating keys or encrypting the file still leaves durable secrets in an artifact, and user data is even more exposed because it is readable via instance metadata — on this exam, any option that keeps access keys on an instance loses to a role.

**How to reason it out**

1. Create an IAM role with the S3 permissions the application needs.
2. Attach the role to the EC2 instance as an instance profile.
3. Update the application to use the default credential chain instead of the configuration file.
4. Verify the application works, then deactivate and delete the IAM user's access keys.

> **Exam tip:** Replace access keys on EC2 with an instance profile role — short-lived credentials beat any scheme for protecting long-term keys.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 15

*Security and Compliance*

**A company is rolling out a three-tier data classification scheme (public, internal, confidential) across hundreds of S3 buckets, EBS volumes, and RDS instances. The operations team must be able to apply tier-appropriate controls and audit classification coverage at scale. Which approach should the team use?** Choose one.

- **a.** Record each resource's classification in a spreadsheet maintained by the security team.
- **b.** Apply a consistent DataClassification tag to every resource and use it to scope policies and AWS Config rules.
- **c.** Encrypt every resource with the same AWS managed key so all data is protected equally.
- **d.** Migrate each classification tier into its own dedicated AWS account.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A spreadsheet drifts from reality immediately and cannot be queried by policies, Config rules, or automation — the classification never drives any control.
- **b** ✅ Tags make the classification machine-readable on the resource itself, so controls can be scoped by tier and coverage can be audited with tooling.
- **c** ❌ Uniform encryption is not classification — confidential data needs stronger controls (customer managed keys, tighter policies, monitoring) than public data, and this approach cannot differentiate.
- **d** ❌ Account-level separation is a heavyweight migration for hundreds of existing resources and still does not mark individual resources — tagging is the standard operational mechanism.

**The concept.** A data classification scheme becomes operational when it lives in resource tags — a consistent tag such as DataClassification lets you scope controls per tier and audit coverage automatically.

**Why this is correct.** Tags travel with the resource and are readable by IAM policies, AWS Config rules, and audit tooling, so tier-appropriate controls follow the classification. A spreadsheet is unenforceable, uniform encryption ignores the tiers entirely, and migrating hundreds of resources into per-tier accounts is enormous overhead that still leaves resources unmarked.

**How to reason it out**

1. Define the classification tiers and the tag key and allowed values.
2. Tag every bucket, volume, and database with its tier.
3. Scope AWS Config rules and policies to the confidential tier for stricter controls.
4. Audit for untagged resources so coverage stays complete.

> **Exam tip:** Classification lives in tags — that is what lets policies and Config rules treat each tier differently.

</details>

📖 Learn this topic: [Data Protection on AWS: KMS, ACM, Secrets, and Security Services (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-data-protection-security-services/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 16

*Security and Compliance*

**An IAM role in account A must upload objects directly to an S3 bucket owned by account B, keeping its own identity rather than assuming a role in account B. The role's identity-based policy allows s3:PutObject on the bucket's objects, but every upload fails with AccessDenied. What is the MOST likely cause?** Choose one.

- **a.** Amazon S3 requires both accounts to belong to the same AWS Organization.
- **b.** The role in account A also needs sts:AssumeRole permission for account B.
- **c.** Cross-account S3 access always requires assuming a role in the bucket owner's account.
- **d.** The bucket policy in account B does not include the account A role as an allowed principal.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Organization membership is not required for cross-account S3 access; aws:PrincipalOrgID is an optional condition, not a prerequisite.
- **b** ❌ No role in account B is being assumed in this design, so sts:AssumeRole is irrelevant to the failure.
- **c** ❌ A resource-based bucket policy can grant a foreign principal direct access while it keeps its own identity — role assumption is one option, not a requirement.
- **d** ✅ Cross-account access requires an allow on both sides: the caller's identity policy in account A and a resource-based bucket policy in account B naming the caller. The missing bucket-policy allow explains the denial.

**The concept.** Within one account an allow from either an identity-based or a resource-based policy is enough; across accounts, both sides must allow — the caller's identity policy and the target resource's policy.

**Why this is correct.** The identity side is already satisfied, so the missing piece is the bucket policy in account B naming the account A role (or its account) as a principal. Role assumption is an alternative pattern, not a requirement, so options invoking sts:AssumeRole are wrong, and there is no organization-membership prerequisite for cross-account S3 access.

**How to reason it out**

1. Confirm the account A role's identity policy allows s3:PutObject on arn:aws:s3:::bucket/*.
2. Add a bucket policy statement in account B allowing s3:PutObject for the account A role's ARN.
3. Check for explicit denies in the bucket policy that could still override the allow.
4. Retry the upload and confirm success in CloudTrail.

> **Exam tip:** Cross-account access needs an allow on both sides — the caller's identity policy and the resource-based policy on the target.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 17

*Networking and Content Delivery*

**A CloudOps engineer is designing subnets for a new VPC and allocates a 10.0.4.0/28 CIDR block to a small management subnet. How many usable IP addresses will be available for resources in this subnet?** Choose one.

- **a.** 11
- **b.** 14
- **c.** 16
- **d.** 13

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ AWS reserves 5 IPs in every subnet — the network address, VPC router, DNS resolver, one for future use, and the broadcast address — leaving 16 − 5 = 11 usable.
- **b** ❌ Subtracting only the network and broadcast addresses is traditional networking math; AWS reserves five addresses per subnet, not two.
- **c** ❌ A /28 contains 16 total addresses, but AWS reserves 5 of them in every subnet, so 16 are never all usable.
- **d** ❌ This assumes only three reserved addresses; AWS holds back five, including the +.3 future-use address and the broadcast address.

**The concept.** In every VPC subnet, AWS reserves five IP addresses that cannot be assigned to resources: the network address, the VPC router (+1), the Amazon DNS resolver (+2), a future-use address (+3), and the broadcast address (the last address, held back even though broadcast is not supported).

**Why this is correct.** A /28 block has 16 total addresses, and after AWS's five reservations, 11 remain assignable. The answer 14 applies traditional networking (network + broadcast reserved) but misses the three AWS-specific reservations, 16 ignores reservations entirely, and 13 undercounts the reserved set. This matters operationally because small subnets fill up fast — load balancer nodes and interface endpoints each consume addresses too.

**How to reason it out**

1. Compute the total addresses in the CIDR: a /28 yields 2^(32−28) = 16.
2. Subtract the five AWS-reserved addresses (network, router, DNS, future use, broadcast).
3. Confirm the result: 16 − 5 = 11 usable addresses.
4. When sizing subnets, add headroom for ENIs consumed by load balancers, endpoints, and NAT gateways.

> **Exam tip:** Every AWS subnet loses exactly five IP addresses to reservations — a /28 gives 11 usable addresses, not 14 or 16.

</details>

📖 Learn this topic: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 18

*Networking and Content Delivery*

**An operations team runs IPv6-enabled EC2 instances in private subnets that must download packages from internet repositories over IPv6. The security policy requires that no connection can be initiated from the internet to these instances. What should the team configure?** Choose one.

- **a.** Route ::/0 to a NAT gateway in a public subnet
- **b.** Route ::/0 to the VPC's internet gateway
- **c.** Create an egress-only internet gateway and route ::/0 to it
- **d.** Create a gateway VPC endpoint for the package repositories

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ NAT gateways perform network address translation for IPv4; there is no IPv6 NAT because VPC IPv6 addresses are globally unique and publicly routable.
- **b** ❌ An internet gateway allows inbound-initiated connections to the instances' publicly routable IPv6 addresses, violating the policy.
- **c** ✅ An egress-only internet gateway is stateful and allows outbound-initiated IPv6 connections and their replies while blocking all inbound-initiated traffic — exactly the outbound-only IPv6 requirement.
- **d** ❌ Gateway endpoints support only Amazon S3 and DynamoDB; they cannot provide general outbound access to internet repositories.

**The concept.** VPC IPv6 addresses are globally unique and publicly routable, so IPv6 has no NAT. The egress-only internet gateway is the IPv6 construct for outbound-only access: instances can initiate connections out, and all inbound-initiated traffic is blocked.

**Why this is correct.** Routing ::/0 to an egress-only internet gateway gives the instances outbound IPv6 access while its stateful behavior blocks anything the internet initiates. A NAT gateway is wrong because it handles IPv4 only; a standard internet gateway is wrong because it exposes the publicly routable IPv6 addresses to inbound connections; and a gateway endpoint only reaches S3 and DynamoDB, not internet repositories.

**How to reason it out**

1. Create an egress-only internet gateway and attach it to the VPC.
2. In the private subnets' route tables, add a route sending ::/0 to the egress-only internet gateway.
3. Verify security groups and NACLs permit the outbound traffic and (for NACLs) the ephemeral-port return traffic.
4. Confirm instances can reach the repositories while inbound IPv6 connection attempts fail.

> **Exam tip:** "IPv6, outbound only" always maps to an egress-only internet gateway — there is no IPv6 NAT.

</details>

📖 Learn this topic: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 19

*Networking and Content Delivery*

**A company wants its bare domain, example.com (the zone apex), to point at a CloudFront distribution. When a CloudOps engineer tries to create a CNAME record at the apex, Route 53 rejects it. What should the engineer create instead?** Choose one.

- **a.** A TXT record containing the distribution's domain name
- **b.** A CNAME on www.example.com and abandonment of the bare domain
- **c.** An MX record with the distribution as the lowest-priority target
- **d.** An alias A record at the apex targeting the CloudFront distribution

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ TXT records hold arbitrary text for uses like domain verification and email policy — they do not route traffic anywhere.
- **b** ❌ Moving to www dodges the problem instead of meeting the requirement — the alias record serves the bare domain directly.
- **c** ❌ MX records designate mail servers for a domain; they have nothing to do with web traffic routing.
- **d** ✅ Alias records are Route 53's extension that points directly at AWS resources and, unlike CNAMEs, works at the zone apex.

**The concept.** DNS forbids a CNAME at the zone apex because it cannot coexist with the SOA and NS records that must live there. Route 53's alias record solves this: it points directly at AWS resources such as CloudFront distributions and load balancers, and it is valid at the apex.

**Why this is correct.** An alias A record at the apex targeting the distribution is exactly the supported pattern for "bare domain to an AWS resource." TXT and MX records serve verification and mail delivery respectively, not traffic routing, and retreating to www abandons the stated requirement rather than fulfilling it.

**How to reason it out**

1. Open the public hosted zone for example.com.
2. Create an A record at the apex and enable the alias option.
3. Select the CloudFront distribution as the alias target.
4. Confirm the apex resolves to the distribution's edge addresses.

> **Exam tip:** "Point the bare domain at an AWS resource" is an alias record every time — CNAMEs are illegal at the zone apex.

</details>

📖 Learn this topic: [Route 53 DNS and CloudFront Content Delivery for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-route53-dns-cloudfront-delivery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 20

*Networking and Content Delivery*

**A CloudOps engineer deploys an application into private subnets of a new VPC. The instances cannot download OS patches from the internet. The private subnets' route tables send 0.0.0.0/0 to a NAT gateway, but the engineer discovers that the NAT gateway was created in a subnet whose route table contains no route to the internet gateway. What should the engineer do to restore outbound internet access?** Choose one.

- **a.** Attach an egress-only internet gateway to the VPC and route 0.0.0.0/0 to it.
- **b.** Create a new NAT gateway in a public subnet whose route table sends 0.0.0.0/0 to the internet gateway, and update the private subnets' route tables to use it.
- **c.** Add a 0.0.0.0/0 route to the internet gateway in the private subnets' route tables.
- **d.** Assign public IPv4 addresses to the instances in the private subnets.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ An egress-only internet gateway handles outbound-only IPv6 traffic (via a ::/0 route); it does not carry IPv4 traffic, which is what OS patch downloads use here.
- **b** ✅ A NAT gateway must sit in a public subnet so it has a path to the internet gateway; a NAT gateway in a private subnet has no way to forward traffic out, so everything behind it silently fails.
- **c** ❌ That would turn the private subnets into public subnets, and the instances still have no public IPs for the internet gateway's one-to-one NAT, so connectivity would still fail.
- **d** ❌ A public IP is useless without a route to an internet gateway in the instance's own subnet, and adding one would defeat the private-subnet design rather than fix the NAT path.

**The concept.** A NAT gateway only works when it is placed in a public subnet — one whose route table sends 0.0.0.0/0 to the internet gateway — because the NAT gateway itself needs that path to forward traffic to the internet on behalf of private instances.

**Why this is correct.** The private subnets' routes are correct, but the NAT gateway's own subnet is effectively private, so the NAT gateway has no path out and drops everything. Recreating it in a genuine public subnet fixes the chain: private subnet routes 0.0.0.0/0 to the NAT gateway, and the NAT gateway's subnet routes 0.0.0.0/0 to the internet gateway. Assigning public IPs to the instances does nothing without an IGW route in their subnet and breaks the private design; routing the private subnets straight to the IGW makes them public and still fails without public IPs; an egress-only internet gateway is an IPv6-only construct.

**How to reason it out**

1. Confirm the private subnets' route tables send 0.0.0.0/0 to the NAT gateway.
2. Check which subnet hosts the NAT gateway and inspect that subnet's route table.
3. Verify the NAT gateway's subnet routes 0.0.0.0/0 to an internet gateway; if not, the NAT gateway is stranded.
4. Create a NAT gateway in a true public subnet and repoint the private subnets' default routes at it.
5. Retest outbound connectivity and confirm security groups and NACLs allow the traffic and its return.

> **Exam tip:** The NAT gateway lives in a public subnet with an IGW route; the private subnets route 0.0.0.0/0 to the NAT gateway — both halves must be true.

</details>

📖 Learn this topic: [AWS Network Troubleshooting: VPC Flow Logs and the Layered Diagnosis Method](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-network-troubleshooting-vpc-flow-logs/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

[← Back to the SOA-C03 study guide](README.md)
