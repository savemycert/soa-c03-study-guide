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

**A CloudOps team manages 300 running EC2 instances that are already Systems Manager managed nodes. The team must start collecting memory and disk-space metrics. Any later change to the collected metrics must be made in one place and reach every running instance without replacing or rebooting it. Which approach meets these requirements with the LEAST operational overhead?** Choose one.

- **a.** Keep one agent configuration file in an S3 bucket and have each instance's boot script download it.
- **b.** Put the agent install and configuration file in the user data of the launch templates the fleet uses.
- **c.** Run AWS-ConfigureAWSPackage through a State Manager association that targets the fleet's managed nodes.
- **d.** Store the agent configuration in Parameter Store and apply it with the AmazonCloudWatch-ManageAgent document.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ There is one copy of the configuration, but instances read it only when they boot, so the running fleet never picks it up and every later change needs a reboot of all 300 instances.
- **b** ❌ User data runs at first boot of new instances only, so the 300 running instances are not configured and later changes again require replacing instances.
- **c** ❌ This installs and keeps the CloudWatch agent package current on every node, but the document applies no agent configuration, so nothing tells the agent to collect memory or disk metrics.
- **d** ✅ A single configuration stored in Parameter Store is the one source of truth, and Run Command with AmazonCloudWatch-ManageAgent (after AWS-ConfigureAWSPackage installs the agent) pushes it to every running managed node; a later change is a parameter update plus one more command run.

**The concept.** The CloudWatch agent reads a JSON configuration. Storing that configuration in Systems Manager Parameter Store and applying it with Systems Manager (AWS-ConfigureAWSPackage to install, AmazonCloudWatch-ManageAgent to configure) gives a whole fleet one consistent, centrally changed configuration.

**Why this is correct.** Three constraints decide it: the instances are already running, they must not be replaced or rebooted, and later changes must be made in one place. Parameter Store plus the AmazonCloudWatch-ManageAgent document meets all three; re-running the command after editing the parameter updates every node in place. AWS-ConfigureAWSPackage is the near-twin: it installs the agent but does not configure it. A configuration file read at boot and launch-template user data only take effect when an instance starts.

**How to reason it out**

1. Note that in-guest metrics (memory, disk space) need the CloudWatch agent with a configuration that lists them.
2. Note the constraints: running instances, no replacement or reboot, one place for later changes.
3. Eliminate boot-time mechanisms (user data, a file downloaded at startup) because running instances never read them.
4. Eliminate AWS-ConfigureAWSPackage on its own because it installs the agent but applies no configuration.
5. Choose Parameter Store for the configuration and AmazonCloudWatch-ManageAgent to apply it.

> **Exam tip:** Fleet-wide CloudWatch agent: configuration in Parameter Store, installed and applied with Systems Manager.

</details>

📖 Learn this topic: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 3

*Monitoring, Logging, Analysis, Remediation, and Performance Optimization*

**A single stateful EC2 instance runs a licensing service that clients reach by its private IP address. The instance becomes unreachable, and the AWS Health Dashboard reports a hardware fault on the physical host beneath it. The team wants the instance brought back automatically with the same identity. Which configuration achieves this?** Choose one.

- **a.** An alarm on StatusCheckFailed_Instance with the EC2 reboot action
- **b.** An Auto Scaling group with minimum, maximum and desired size 1
- **c.** An alarm on StatusCheckFailed_System with the EC2 reboot action
- **d.** An alarm on StatusCheckFailed_System with the EC2 recover action

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ This watches the check that covers the guest OS, and a reboot keeps the instance on the same faulty host, so it cannot cure a hardware problem.
- **b** ❌ Auto Scaling terminates the unhealthy instance and launches a new one with a new instance ID and private IP, which breaks clients that use the old address.
- **c** ❌ This watches the right check, but a reboot restarts the instance on the same failed host. Only recover (or a stop and start) moves it to new hardware.
- **d** ✅ A host-level fault fails the system status check, and the recover action moves the instance to healthy hardware while keeping its instance ID, private and Elastic IP addresses and EBS volumes.

**The concept.** EC2 reports two status checks. The system status check (StatusCheckFailed_System) covers the AWS infrastructure under the instance: host hardware, power and network. The instance status check (StatusCheckFailed_Instance) covers the guest OS. The recover alarm action works with the system check and moves the instance to a different host.

**Why this is correct.** A hardware fault on the host is an infrastructure failure, so the system status check is the right metric, and the recover action is the fix: the instance restarts on healthy hardware with the same instance ID, private IP, Elastic IP and EBS volumes. Pairing the right metric with the reboot action leaves the instance on the broken host; pairing the instance check with reboot watches the wrong layer; Auto Scaling replaces the instance with a new identity.

**How to reason it out**

1. Classify the failure: a host hardware fault is AWS infrastructure, which the system status check covers.
2. Pick the action that changes the host: recover (reboot stays on the same host).
3. Check the identity requirement: recovery keeps the instance ID and IP addresses, while Auto Scaling replacement does not.

> **Exam tip:** System check failed + keep the same instance = alarm on StatusCheckFailed_System with the recover action.

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

- **a.** Step scaling policies with CloudWatch alarms at 60%, 70% and 80% average CPU
- **b.** A simple scaling policy that adds one instance when average CPU exceeds 50%
- **c.** Scheduled actions that raise capacity at the hours when CPU usually peaks
- **d.** A target tracking policy that holds the group's average CPU at 50%

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Step scaling works, but the engineer must design and maintain the alarms and adjustment tiers, and it reacts to threshold breaches rather than holding a value — more overhead for a looser fit.
- **b** ❌ Simple scaling makes one fixed adjustment and then waits out a cooldown, and this policy only scales out. It lags changes in load and never brings CPU back up to the target by removing instances.
- **c** ❌ Scheduled actions change capacity by the clock, not by measured CPU, so they cannot hold the average near 50% when real traffic departs from the usual pattern.
- **d** ✅ Correct. Target tracking needs only the metric and the target value; AWS creates and manages the CloudWatch alarms and scales out and in to hold average CPU near 50%.

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

**An operations team wants an Auto Scaling group to add one instance when average CPU exceeds 70% but add four instances at once when average CPU exceeds 90%, using one policy attached to one CloudWatch alarm. Which scaling policy type can respond this way?** Choose one.

- **a.** Target tracking scaling
- **b.** Scheduled scaling
- **c.** Step scaling
- **d.** Simple scaling

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Target tracking takes a single target value and manages its own alarms; it does not let you define different adjustment sizes for different breach severities.
- **b** ❌ Scheduled scaling changes capacity at specific times on a schedule; it does not react to CPU at all.
- **c** ✅ Step scaling defines step adjustments tied to alarm breach magnitude — +1 instance for the 70–90% band and +4 above 90% — so the response scales with severity.
- **d** ❌ Simple scaling ties one alarm to one fixed adjustment and then waits out a cooldown, so it cannot respond differently to a small breach versus a large one.

**The concept.** Step scaling sizes the capacity adjustment to how far the metric breached the alarm threshold, using step adjustments you define.

**Why this is correct.** Only step scaling lets you say 'a small breach adds one instance, a large breach adds four' — the adjustment scales with breach magnitude. Target tracking holds one value and hides its alarms; simple scaling makes exactly one fixed adjustment per cooldown; scheduled scaling ignores metrics entirely.

**How to reason it out**

1. Identify the requirement: different response sizes at different severity levels of the same metric.
2. Create a CloudWatch alarm on average CPU for the Auto Scaling group.
3. Attach a step scaling policy with step adjustments: +1 instance for the 70–90% band, +4 instances above 90%.
4. Configure instance warmup so newly launched instances are counted correctly during multi-step responses.

> **Exam tip:** Adjustments sized to how far the alarm is breached are the signature use case for step scaling.

</details>

📖 Learn this topic: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 7

*Reliability and Business Continuity*

**A company runs a multiplayer game backend on EC2 instances that exchange game state with players over UDP. Several partner studios must add the backend's addresses to their firewall allowlists, and their policy requires one fixed IP address in each Availability Zone the service uses. Which load balancer configuration should a CloudOps engineer deploy?** Choose one.

- **a.** A Network Load Balancer with a UDP listener and an Elastic IP address assigned in each enabled AZ
- **b.** A Gateway Load Balancer forwarding the UDP game port, with an Elastic IP address in each enabled AZ
- **c.** An Application Load Balancer behind AWS Global Accelerator, with a UDP listener on the accelerator
- **d.** An internet-facing Classic Load Balancer with a TCP listener on the game port in each enabled AZ

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The NLB is a layer 4 load balancer that supports UDP listeners, and it gets one static IP address per enabled Availability Zone, to which you can assign your own Elastic IP addresses. That satisfies both the protocol and the allowlist requirement.
- **b** ❌ A Gateway Load Balancer inserts third-party virtual appliances (firewalls, IDS) into a traffic path using GENEVE encapsulation on port 6081. It is not a client-facing load balancer, has no per-port listeners for clients and does not take Elastic IP addresses.
- **c** ❌ Global Accelerator does offer UDP listeners and static IP addresses, but its two anycast addresses are global rather than one per AZ, and an ALB endpoint only receives HTTP and HTTPS over TCP, so the UDP game traffic cannot reach it.
- **d** ❌ The Classic Load Balancer supports only TCP, SSL, HTTP and HTTPS listeners, so it cannot carry UDP game traffic, and it exposes a DNS name whose addresses change rather than a fixed IP per AZ.

**The concept.** Elastic Load Balancing offers different load balancer types for different layers. The Network Load Balancer works at layer 4 (TCP, UDP, TCP_UDP, TLS) and gives each enabled Availability Zone a static IP address that can be one of your Elastic IP addresses. The Application Load Balancer works at layer 7 (HTTP/HTTPS) and is reached only through a DNS name.

**Why this is correct.** Two requirements decide this scenario: UDP traffic and one fixed address per AZ for allowlisting. Only the Network Load Balancer meets both, and assigning an Elastic IP in each AZ keeps those addresses under the company's control. Global Accelerator is the tempting alternative because it is famous for static IPs, but its two anycast addresses are global rather than per AZ, and an ALB endpoint still cannot accept UDP. The Gateway Load Balancer sounds layer-3/4 capable but exists for transparent appliance insertion, not for serving clients. The Classic Load Balancer is a legacy option with no UDP support.

**How to reason it out**

1. Identify the protocol: UDP rules out the Application Load Balancer and the Classic Load Balancer.
2. Identify the addressing requirement: one fixed IP per AZ points to the NLB's static per-AZ addresses.
3. Rule out the Gateway Load Balancer, which is for inserting virtual appliances rather than serving clients.
4. Create the NLB with a UDP listener and assign an Elastic IP address in each enabled AZ, then share those addresses with the partners.

> **Exam tip:** UDP or a fixed IP per Availability Zone means a Network Load Balancer; Global Accelerator gives global anycast IPs, not per-AZ ones.

</details>

📖 Learn this topic: [High Availability: ELB and Route 53 Health Checks, Multi-AZ Design](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-high-availability-health-checks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 8

*Reliability and Business Continuity*

**After a post-incident review, company leadership mandates that the order-processing workload must experience no more than 30 minutes of data loss and must be back online within 2 hours of any failure. A CloudOps engineer is recording these targets in the disaster recovery plan. How should the engineer document them?** Choose one.

- **a.** Two RPO targets: a 30-minute RPO for transactions and a 2-hour RPO for the service
- **b.** RPO of 30 minutes and RTO of 2 hours
- **c.** RTO of 30 minutes and RPO of 2 hours
- **d.** Two RTO targets: a 30-minute RTO for data and a 2-hour RTO for the application

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ RPO measures acceptable data loss only. The 2-hour limit on how long the service may be down is a downtime target, which is the RTO.
- **b** ✅ Maximum acceptable data loss expressed as time is the recovery point objective (RPO), and maximum acceptable downtime is the recovery time objective (RTO). "No more than 30 minutes of data loss" is the RPO; "back online within 2 hours" is the RTO.
- **c** ❌ This reverses the two objectives. The 30-minute figure describes data loss, which maps to RPO, and the 2-hour figure describes downtime, which maps to RTO.
- **d** ❌ RTO measures downtime only. A limit on how much data may be lost is a separate objective — the RPO — not a second flavor of RTO.

**The concept.** RTO is the maximum acceptable time between failure and full restoration of service; RPO is the maximum acceptable data loss measured as time — how old the most recent recoverable copy may be.

**Why this is correct.** The 30-minute data-loss limit is the RPO and the 2-hour recovery deadline is the RTO. Swapping the two is the classic trap. Recording two RTOs or two RPOs collapses both requirements into one objective type, but downtime and data loss are independent: a fast restore of an old backup meets the RTO while missing the RPO, and a fresh copy that takes too long to restore does the opposite.

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

**During an account cleanup, a CloudOps engineer deregistered dozens of outdated AMIs that had been shared with two other accounts. The next monthly bill still shows the same EBS snapshot storage charges. What explains the charges?** Choose one.

- **a.** The snapshots that backed the AMIs stay in the account until they are deleted separately.
- **b.** The snapshots are held for a fixed retention period before EC2 deletes them by itself.
- **c.** The snapshots stay billed to the owner because the AMIs had been shared with other accounts.
- **d.** The snapshots keep billing until the EBS volumes they were taken from are deleted too.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Deregistering an AMI only removes the ability to launch from it; the EBS snapshots behind it remain and keep accruing storage charges until someone deletes them.
- **b** ❌ EC2 has no automatic snapshot cleanup tied to AMI deregistration; without a separate deletion (manual or a lifecycle policy) the snapshots persist indefinitely.
- **c** ❌ Sharing grants launch permissions only; it does not change snapshot lifecycle. The charges would continue whether or not the AMIs had been shared.
- **d** ❌ Snapshots are billed independently of their source volumes; deleting or keeping the volumes does not stop snapshot storage charges.

**The concept.** Deregistering an AMI and deleting its backing EBS snapshots are two separate operations, so image cleanup is always a two-step job.

**Why this is correct.** Deregistration only prevents new launches from the image; the EBS snapshots referenced in its block device mappings stay in the account and keep billing until deleted. There is no built-in retention countdown that removes them, sharing affects launch permissions rather than storage, and snapshots do not depend on their source volumes for billing.

**How to reason it out**

1. Before deregistering, record the snapshot IDs in each AMI's block device mappings.
2. Deregister the AMI so no new instances can launch from it.
3. Delete the orphaned snapshots explicitly (or let an Amazon Data Lifecycle Manager policy handle AMI deprecation and snapshot deletion).
4. Confirm in Cost Explorer that EBS snapshot storage charges fall.

> **Exam tip:** Deregistering an AMI never deletes its snapshots - delete them separately or keep paying for them.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 10

*Deployment, Provisioning, and Automation*

**A company's disaster recovery plan requires launching its hardened web server image in a second Region within minutes of a decision to fail over. The AMI currently exists only in us-east-1, and the DR Region's launch template is still empty. What should a CloudOps engineer do now to meet the plan?** Choose one.

- **a.** Copy the AMI from us-east-1 into the DR Region now and reference the new AMI ID there.
- **b.** Copy the AMI's backing snapshots into the DR Region and reference them in the launch template.
- **c.** Wait for the failover decision, then copy the AMI into the DR Region and launch from it.
- **d.** Reference the us-east-1 AMI ID directly in the launch template created in the DR Region.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Copying the AMI creates a new AMI (with its own ID and snapshots) in the DR Region, which the DR launch template can then reference.
- **b** ❌ A launch template needs an AMI ID as its image; snapshots alone are not a launchable image until someone registers an AMI from them, which adds a manual step at failover.
- **c** ❌ Copying an AMI copies all of its snapshot data and can take a long time for large images, so starting the copy at failover time risks missing the within-minutes requirement.
- **d** ❌ AMI IDs are Regional; a launch template in another Region cannot resolve an AMI that exists only in us-east-1, so launches would fail.

**The concept.** AMIs are Regional resources: to launch an image in another Region you copy it there, which creates a new AMI with a new ID.

**Why this is correct.** An AMI and its snapshots live in one Region, so launching the same image elsewhere needs a copy there, and the copy gets a new AMI ID. Because a copy moves all the snapshot data and can take a long time, it must be done ahead of a failover, not at the moment of it. A foreign AMI ID does not resolve in another Region, and copied snapshots are not a launchable image until an AMI is registered from them.

**How to reason it out**

1. Recognise that AMI IDs are scoped to a single Region.
2. Note the time constraint: a copy is not instant, so it must exist before failover.
3. Copy the AMI into the DR Region now (CopyImage, or an EC2 Image Builder distribution setting for every new build).
4. Reference the new Regional AMI ID in the DR launch template and re-copy whenever the golden image changes.

> **Exam tip:** One AMI, one Region - copy it to every Region that must launch it.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 11

*Deployment, Provisioning, and Automation*

**A security mandate bans SSH key pairs, bastion hosts and any inbound administration port on production EC2 instances. Engineers still need interactive command-line access to the instances, and auditors want a record of every command typed. The instances are already managed by Systems Manager. Which access method meets the requirements?** Choose one.

- **a.** Connect through an EC2 Instance Connect Endpoint in the private subnet, with CloudTrail recording each connection
- **b.** Start Session Manager sessions with the AWS-StartSSHSession document, with session logging to S3
- **c.** Start Session Manager shell sessions from the console or CLI, with session logging to CloudWatch Logs
- **d.** Connect through the EC2 serial console, with CloudTrail recording each serial console key push

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ An Instance Connect Endpoint removes the need for a public IP or bastion, but it is still SSH: a short-lived key is pushed and the instance must accept inbound port 22 from the endpoint. CloudTrail records the connection, not the commands typed.
- **b** ❌ Tunnelling SSH through Session Manager still needs SSH key pairs, and Session Manager cannot log sessions that connect through SSH or port forwarding because the session data is encrypted end to end inside SSH.
- **c** ✅ A standard Session Manager shell rides the SSM Agent's outbound connection, so it needs no keys, no bastion and no inbound port, and session logging captures the commands and output for auditors.
- **d** ❌ Serial console access also pushes an SSH public key and needs an OS user with a password; CloudTrail captures the API call that opens the connection but nothing that is typed in it.

**The concept.** Session Manager provides interactive shell access without SSH keys, bastion hosts or inbound ports because sessions are brokered over the SSM Agent's outbound connection; standard shell sessions can be logged to CloudWatch Logs or S3, but SSH and port-forwarding sessions cannot.

**Why this is correct.** Only a standard Session Manager shell session meets every constraint: IAM controls who may start it, nothing listens inbound, and session logging records each command. The tempting near-twin is the AWS-StartSSHSession tunnel, which looks like Session Manager but still uses SSH keys and is excluded from session logging. Instance Connect Endpoint and the serial console both push SSH keys, and CloudTrail only shows that a connection was opened.

**How to reason it out**

1. List the constraints: no keys, no bastion, no inbound port, command-level audit.
2. Eliminate every option that pushes or uses an SSH key, including SSH tunnelled through Session Manager.
3. Separate API-level records (CloudTrail) from session content records (Session Manager logging).
4. Pick the standard Session Manager shell with logging to CloudWatch Logs or S3.

> **Exam tip:** No keys, no inbound ports and a log of every command means a standard Session Manager shell with session logging; SSH tunnelled through it is not logged.

</details>

📖 Learn this topic: [Systems Manager and event-driven automation: managing resources for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-systems-manager-event-driven-automation/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 12

*Deployment, Provisioning, and Automation*

**A security team has approved a hardened base AMI in its tooling account. Two specific workload accounts in the same organization need to launch instances from it, and no other account in the organization may see it. What should a CloudOps engineer do with the LEAST operational effort?** Choose one.

- **a.** Add the two account IDs to the AMI's launch permissions in the tooling account.
- **b.** Run a copy-image operation from the tooling account into both of the workload accounts.
- **c.** Share the AMI's backing snapshots with both workload accounts through snapshot permissions.
- **d.** Share the AMI with the organization so that both workload accounts can launch it.

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Adding specific account IDs to an AMI's launch permissions shares it privately with exactly those accounts, which is the intended mechanism.
- **b** ❌ CopyImage creates the copy in the caller's own account; it cannot push an AMI into another account. The target accounts would first need launch access to copy it themselves.
- **c** ❌ Snapshot permissions let the accounts create volumes from the snapshots or register their own AMIs from them, which is extra work per account and leaves separate images to manage; it does not share the AMI itself.
- **d** ❌ Sharing with the organization's ARN would expose the AMI to every member account, which breaks the requirement that no other account can see it.

**The concept.** A private AMI is shared by adding account IDs (or an organization/OU ARN) to its launch permissions; scope the share to exactly the principals that need it.

**Why this is correct.** Launch permissions are the AMI's own sharing control and can name individual accounts, an OU or the whole organization. Naming the two account IDs gives access to them and nobody else in one step. Organization-wide sharing breaks the visibility constraint, CopyImage only creates copies in the calling account, and sharing only the snapshots makes each account rebuild its own AMI. If the AMI's snapshots are encrypted with a customer managed KMS key, the key policy must also let those accounts use the key, or launches fail.

**How to reason it out**

1. Identify the narrowest principal set that needs access: here, two account IDs.
2. Add those account IDs to the AMI's launch permissions (ModifyImageAttribute).
3. If the snapshots use a customer managed KMS key, grant the accounts kms:Decrypt/CreateGrant on that key.
4. Have the workload accounts launch from the AMI ID, or copy it to own a local copy.

> **Exam tip:** Private AMI sharing = launch permissions, scoped to the exact accounts or OUs; encrypted AMIs also need the key shared.

</details>

📖 Learn this topic: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 13

*Security and Compliance*

**A security audit finds IAM users in an AWS account who call AWS APIs with access keys and sign in to the console without a second factor. The auditor requires that these users be unable to do anything except set up their own MFA device until they authenticate with one. The administrator has already reviewed the account password policy. What should the administrator do?** Choose one.

- **a.** Attach a group policy that denies actions except MFA self-management when aws:MultiFactorAuthAge exceeds 3600 seconds.
- **b.** Attach a group policy that denies actions except MFA self-management when BoolIfExists aws:MultiFactorAuthPresent is false.
- **c.** Require MFA in the IAM account password policy and set passwords to expire after 90 days.
- **d.** Deploy the AWS Config managed rule iam-user-mfa-enabled with an SSM Automation remediation for users it flags.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ aws:MultiFactorAuthAge is absent from requests made without MFA, and a numeric condition on a missing key evaluates to false, so the deny never applies to exactly the users the auditor is worried about.
- **b** ✅ This is the AWS-documented pattern. BoolIfExists also matches requests that carry no MFA context at all, such as long-term access-key calls, so those users can do nothing except register an MFA device until they sign in with it.
- **c** ❌ The account password policy controls console password rules such as length, character types, reuse and expiry. It has no MFA setting, so this cannot enforce the requirement.
- **d** ❌ The rule detects users without an MFA device after the fact. A remediation runbook cannot register a device on a user's behalf, and nothing stops those users from calling APIs in the meantime.

**The concept.** The IAM account password policy cannot require MFA. MFA is enforced with an identity policy that denies everything except MFA self-management when the request lacks MFA context.

**Why this is correct.** A Deny statement conditioned on BoolIfExists aws:MultiFactorAuthPresent = false matches both console sessions without MFA and access-key requests, which carry no MFA key at all, so the users are locked out until they authenticate with a device. The password policy has no MFA option. The MultiFactorAuthAge variant looks equivalent but fails open: the key is missing when MFA was not used, so the condition is false and nothing is denied. AWS Config only reports users without a device.

**How to reason it out**

1. Rule out the password policy: it only governs console password rules.
2. Rule out detective tools (Config rules) when the requirement is to block actions.
3. Write a Deny with an exception for the MFA self-management actions.
4. Use BoolIfExists on aws:MultiFactorAuthPresent so requests without any MFA context are also denied.

> **Exam tip:** Enforce MFA with a Deny on BoolIfExists aws:MultiFactorAuthPresent = false; the password policy has no MFA switch.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 14

*Security and Compliance*

**During a credential review, a CloudOps engineer finds that an application on an EC2 instance reads an IAM user's access key ID and secret access key from a configuration file to call Amazon S3. The security team wants no long-term credentials on the instance and no change to how the AWS SDK is configured in the code. What should the engineer do?** Choose one.

- **a.** Encrypt the configuration file with a KMS key and grant the instance permission to decrypt it at startup.
- **b.** Attach an IAM role through an instance profile, then delete the file and the IAM user's access keys.
- **c.** Move the access keys into environment variables that the instance's user data script exports at boot.
- **d.** Store the access keys in Secrets Manager with rotation enabled and have the application fetch them at startup.

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Encryption protects the file at rest, but the application still decrypts and uses the same long-term keys, and it needs new decryption code.
- **b** ✅ The SDK's default credential provider chain picks up the role's temporary, automatically rotated credentials from the instance metadata service, so no long-term keys remain and the code is unchanged.
- **c** ❌ The SDK would read them without a code change, but they are still long-term keys, and user data is readable by anyone who can view the instance attributes.
- **d** ❌ The keys are still long-term IAM user credentials, and the application now needs code to fetch them from Secrets Manager, which breaks the no-code-change constraint.

**The concept.** Workloads on EC2 should get credentials from an IAM role delivered through an instance profile, not from long-term access keys stored on the instance.

**Why this is correct.** An instance profile makes the role's temporary credentials available through the instance metadata service, and every AWS SDK's default credential provider chain finds them without code changes. Deleting the IAM user's keys removes the long-term secret entirely. Secrets Manager and KMS protect the keys but keep them long-term and need code changes; environment variables via user data keep long-term keys in a readable place.

**How to reason it out**

1. Identify the risk: long-term IAM user keys on the instance.
2. Prefer temporary credentials from an IAM role for any AWS-hosted workload.
3. For EC2, attach the role through an instance profile; the SDK finds it automatically.
4. Remove the file and delete the user's access keys.

> **Exam tip:** Workloads on EC2 use an instance profile role, never stored access keys.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 15

*Security and Compliance*

**A company is rolling out a three-tier data classification scheme (public, internal, confidential) across hundreds of S3 buckets, EBS volumes and RDS instances. The operations team must apply stricter controls to the confidential tier and produce an automated report of resources that have not been classified yet. Which approach should the team implement?** Choose one.

- **a.** Encrypt confidential resources with a dedicated customer managed KMS key and report on key use in AWS Config
- **b.** Apply a DataTier tag to every resource and scope AWS Config rules and IAM conditions to its value
- **c.** Group each classification tier's resources into AWS Resource Groups by the CloudFormation stacks that created them
- **d.** Run Macie automated sensitive data discovery and use each resource's sensitivity score as its tier

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A dedicated key is a sensible control for the confidential tier, but it marks only one tier, cannot distinguish public from internal data, and gives AWS Config nothing to check for resources that were never classified.
- **b** ✅ A tag stores the tier on the resource itself. Config rules can be scoped by tag, IAM policies can use aws:ResourceTag conditions, and the required-tags managed rule reports resources that are missing the tag.
- **c** ❌ Stack-based resource groups mirror how resources were deployed, not how sensitive their data is, and resources created outside CloudFormation never appear. Nothing on the resource records its tier for policies or Config rules to read.
- **d** ❌ Macie only analyzes S3 objects, so EBS volumes and RDS instances would get no tier at all. Macie is the tool that checks whether S3 tiers are right, not the mechanism that records them.

**The concept.** A data classification scheme becomes enforceable when the tier is stored on each resource as a consistent tag that policies, AWS Config rules and reports can read.

**Why this is correct.** A tag such as DataTier travels with the resource and are understood by IAM (aws:ResourceTag conditions), AWS Config (rule scope by tag, and the required-tags managed rule to find untagged resources) and cost and inventory tooling. Stack-based resource groups describe deployment, not sensitivity. Macie is limited to S3. A dedicated KMS key is a control for one tier, not a marker that covers all three.

**How to reason it out**

1. Identify the two needs: per-tier controls and a coverage report across several resource types.
2. Ask which mechanism stores the tier on every resource type in the stem.
3. Check that IAM conditions and Config rules can read that mechanism.
4. Reject options limited to S3 or to a single tier.

> **Exam tip:** Classification lives in tags; controls and coverage checks read the tag.

</details>

📖 Learn this topic: [Data Protection on AWS: KMS, ACM, Secrets, and Security Services (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-data-protection-security-services/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 16

*Security and Compliance*

**An IAM role in account A must upload objects directly to an S3 bucket owned by account B while keeping its own identity, without assuming a role in account B. The role's identity-based policy allows s3:PutObject on arn:aws:s3:::reports-b/*, the bucket uses SSE-S3 and has S3 Block Public Access turned on. Every upload fails with AccessDenied. What is the MOST likely cause?** Choose one.

- **a.** The identity-based policy also needs s3:PutObjectAcl so that ownership of uploads passes to account B.
- **b.** S3 Block Public Access on the bucket rejects requests that come from principals in other AWS accounts.
- **c.** The role in account A also needs sts:AssumeRole permission on a role in account B before it can upload.
- **d.** The bucket policy in account B has no statement that grants the account A role s3:PutObject.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ With the default Object Ownership setting (bucket owner enforced) ACLs are disabled and the bucket owner owns every object, so no ACL permission is needed for the upload to succeed.
- **b** ❌ Block Public Access blocks public grants (for example Principal "*" without restricting conditions). A bucket policy that names a specific role in another account is not public, so it is not blocked.
- **c** ❌ No role in account B is assumed in this design. Direct cross-account access with the caller's own identity uses the bucket policy, not role assumption.
- **d** ✅ Across accounts both sides must allow: the identity policy in account A and the resource-based bucket policy in account B. The missing bucket-policy grant explains the AccessDenied.

**The concept.** Within one account, an allow from either the identity-based or the resource-based policy is enough. Across accounts, both the caller's identity policy and the target resource's policy must allow the request.

**Why this is correct.** The identity policy in account A already allows s3:PutObject, so the gap is on account B's side: the bucket policy must name the role (or account A) as a principal. Role assumption is an alternative design, not a requirement. Block Public Access does not affect a policy that grants a specific external principal, and with ACLs disabled by default there is no object-ownership step.

**How to reason it out**

1. Note that the caller keeps its own identity, so this is resource-policy based cross-account access.
2. Remember that cross-account requests need an allow in both accounts.
3. Confirm account A's side already allows the action.
4. Conclude the bucket policy in account B is missing the grant.

> **Exam tip:** Cross-account access needs two allows: the caller's identity policy and the resource's policy.

</details>

📖 Learn this topic: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 17

*Networking and Content Delivery*

**A CloudOps engineer is designing subnets for a new VPC and allocates a 10.0.4.0/28 CIDR block to a small management subnet. How many usable IP addresses will be available for resources in this subnet?** Choose one.

- **a.** 11
- **b.** 16
- **c.** 14
- **d.** 13

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ AWS reserves 5 IPs in every subnet — the network address, VPC router, DNS resolver, one for future use, and the broadcast address — leaving 16 − 5 = 11 usable.
- **b** ❌ A /28 contains 16 total addresses, but AWS reserves 5 of them in every subnet, so 16 are never all usable.
- **c** ❌ Subtracting only the network and broadcast addresses is traditional networking math; AWS reserves five addresses per subnet, not two.
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

- **a** ❌ A NAT gateway does not give IPv6 instances general IPv6 internet egress. Its only IPv6 role is NAT64 (with DNS64), which translates traffic to IPv4 destinations through a 64:ff9b::/96 route, not a ::/0 route to IPv6 repositories.
- **b** ❌ An internet gateway allows inbound-initiated connections to the instances' publicly routable IPv6 addresses, violating the policy.
- **c** ✅ An egress-only internet gateway is stateful and allows outbound-initiated IPv6 connections and their replies while blocking all inbound-initiated traffic — exactly the outbound-only IPv6 requirement.
- **d** ❌ Gateway endpoints support only Amazon S3 and DynamoDB; they cannot provide general outbound access to internet repositories.

**The concept.** VPC IPv6 addresses are globally unique and publicly routable, so there is no NAT for IPv6-to-IPv6 traffic. The egress-only internet gateway is the IPv6 construct for outbound-only access: instances can initiate connections out, and inbound-initiated traffic is blocked.

**Why this is correct.** Routing ::/0 to an egress-only internet gateway gives the instances outbound IPv6 access while its stateful behavior blocks anything the internet initiates. A NAT gateway is wrong because its IPv6 support is limited to NAT64 translation toward IPv4 destinations; a standard internet gateway is wrong because it accepts inbound connections to the publicly routable IPv6 addresses; and a gateway endpoint only reaches S3 and DynamoDB, not internet repositories.

**How to reason it out**

1. Create an egress-only internet gateway and attach it to the VPC.
2. In the private subnets' route tables, add a route sending ::/0 to the egress-only internet gateway.
3. Verify security groups and NACLs permit the outbound traffic and (for NACLs) the ephemeral-port return traffic.
4. Confirm instances can reach the repositories while inbound IPv6 connection attempts fail.

> **Exam tip:** "IPv6, outbound only" maps to an egress-only internet gateway; a NAT gateway's NAT64 only helps IPv6 hosts reach IPv4 destinations.

</details>

📖 Learn this topic: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 19

*Networking and Content Delivery*

**A company serves its website through a CloudFront distribution and wants visitors who type the bare domain, example.com, to reach the distribution too. A CloudOps engineer tries to add a CNAME record for example.com in the Route 53 public hosted zone and the change is rejected. The record must keep working when CloudFront's edge addresses change. What should the engineer create?** Choose one.

- **a.** An alias A record for example.com that targets the ALB origin behind the distribution
- **b.** A weighted CNAME record for example.com with a weight of 100 that targets the distribution
- **c.** An A record for example.com listing the IP addresses the distribution's domain name resolves to now
- **d.** An alias A record for example.com that targets the distribution's domain name

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ This is a valid apex record, but it sends visitors straight to the origin and bypasses the distribution's caching and edge features, which breaks the requirement.
- **b** ❌ A routing policy does not change the record type rules: a CNAME cannot exist at the zone apex, whatever policy it uses.
- **c** ❌ CloudFront's edge addresses vary by location and change over time, so a hard-coded A record breaks and also pins every user to a few edge locations.
- **d** ✅ An alias record points at the AWS resource itself, is valid at the zone apex where a CNAME is not, and Route 53 follows the target's addresses automatically.

**The concept.** DNS does not allow a CNAME at the zone apex, because the apex must also hold the SOA and NS records and a CNAME cannot coexist with other records. Route 53 alias records point directly at AWS resources such as CloudFront distributions and load balancers and are allowed at the apex.

**Why this is correct.** An alias A record targeting the distribution meets both needs: it is legal at the apex, and Route 53 answers with the distribution's current addresses, so nothing breaks when CloudFront's edge IPs change. A static A record goes stale, an alias to the ALB skips CloudFront entirely, and wrapping the CNAME in a weighted policy is still a CNAME at the apex.

**How to reason it out**

1. Recognise example.com as the zone apex, where CNAME records are not allowed.
2. Rule out hard-coded IP addresses because CloudFront's edge addresses change.
3. Choose an alias record, which works at the apex and tracks the target.
4. Check the alias target is the distribution itself, not its origin.

> **Exam tip:** Bare domain to an AWS resource means an alias record; a CNAME is never valid at the zone apex, whatever routing policy wraps it.

</details>

📖 Learn this topic: [Route 53 DNS and CloudFront Content Delivery for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-route53-dns-cloudfront-delivery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

## Question 20

*Networking and Content Delivery*

**A CloudOps engineer deploys an application into private subnets, and the instances cannot download OS patches. The private subnets' route tables send 0.0.0.0/0 to a public zonal NAT gateway that is Available. The NAT gateway was created in the database subnet, whose route table holds only the local route, and the database subnet must stay without any internet route. What should the engineer do?** Choose one.

- **a.** Replace the NAT gateway with a private NAT gateway in the same subnet and keep the private routes
- **b.** Create a NAT gateway in a subnet that routes 0.0.0.0/0 to the internet gateway and repoint the private routes to it
- **c.** Assign public IPv4 addresses to the application instances and keep their default route on the NAT gateway
- **d.** Add a 0.0.0.0/0 route to the internet gateway in the route table of the subnet that hosts the NAT gateway

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A private NAT gateway translates traffic toward other VPCs or on-premises networks through a transit gateway or virtual private gateway. It cannot send traffic to the internet through an internet gateway, so patch downloads still fail.
- **b** ✅ A public zonal NAT gateway forwards traffic using its own subnet's route table, so it needs a subnet with a 0.0.0.0/0 route to the internet gateway. Building it in a proper public subnet fixes egress without giving the database subnet an internet route.
- **c** ❌ A public IP on the instance does nothing while its subnet's default route still points at the NAT gateway, and the NAT gateway itself still has no path to the internet gateway.
- **d** ❌ This would make the existing NAT gateway work, but that route table belongs to the database subnet, which the stem says must stay without an internet route.

**The concept.** A public zonal NAT gateway translates private instances' traffic and then forwards it using the route table of the subnet it lives in, so that subnet must route 0.0.0.0/0 to an internet gateway.

**Why this is correct.** The private side of the chain is correct; the broken link is the NAT gateway's own subnet, which has no route to the internet gateway. Two fixes would work: add an internet gateway route to that subnet, or move NAT to a subnet that already has one. The constraint that the database subnet must stay without an internet route rules out the first, so the engineer creates a NAT gateway in a real public subnet and repoints the private routes. Public IPs on the instances do not help while traffic still goes to the NAT gateway, and a private NAT gateway cannot reach the internet.

**How to reason it out**

1. Confirm the private subnets' 0.0.0.0/0 route targets the NAT gateway.
2. Check the route table of the subnet that hosts the NAT gateway for a 0.0.0.0/0 route to the internet gateway.
3. Note that adding that route would expose the database subnet, which the requirement forbids.
4. Create the NAT gateway in a public subnet and update the private route tables to target it.

> **Exam tip:** Private subnet routes to the NAT gateway; the NAT gateway's own subnet routes to the internet gateway. Both links must exist.

</details>

📖 Learn this topic: [AWS Network Troubleshooting: VPC Flow Logs and the Layered Diagnosis Method](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-network-troubleshooting-vpc-flow-logs/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

---

[← Back to the SOA-C03 study guide](README.md)
