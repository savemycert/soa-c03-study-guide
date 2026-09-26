# Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization (22%)

This domain is about seeing what production is doing and acting on it: collecting the right CloudWatch metrics and logs, turning signals into alarms and automated fixes, and reading utilization data to pick the cheapest performance change. It also absorbed the old cost and performance domain, so expect symptom-to-fix questions about EC2, EBS, S3, shared storage, and RDS.

## 1.1 Implement metrics, alarms, and filters by using AWS monitoring and logging services

- **A metric is identified by namespace + name + dimensions.** Each unique combination is its own metric, so querying a custom metric without the dimensions it was published with returns nothing.
- **Resolution:**

| Source | Interval | Notes |
|---|---|---|
| EC2 standard monitoring | 5 minutes, free | Hypervisor view: CPU, network, disk I/O, status checks |
| EC2 detailed monitoring | 1 minute, paid | Same metrics, faster. Adds nothing new |
| Custom metric, standard resolution | 1 minute | Published with `PutMetricData` |
| Custom metric, high resolution | Down to 1 second | Supports alarms on 10- or 30-second periods |
| CloudWatch agent | You configure it | In-guest memory, disk space, swap, processes, plus log files |

- **Top trap:** memory and free disk space are never in the default EC2 metrics. "Memory metrics missing" means install the CloudWatch agent. Keep its JSON config in Parameter Store and roll it out with Systems Manager.
- **Containers:** Container Insights collects cluster, service, task, and pod data for ECS and EKS. Prometheus-format metrics go to Amazon Managed Service for Prometheus, visualized in Managed Grafana. Lambda publishes its own metrics with no agent.
- **Logs:** retention, metric filters, and subscription filters attach to the **log group**; a **log stream** is one source. Retention defaults to never expire, so runaway log cost means set retention or export to S3.
- **Logs Insights** queries data already ingested (for incident digging). It does not create metrics or alarms.
- **Metric filter vs subscription filter:** a metric filter counts pattern matches into a metric you can alarm on, only for data ingested after it exists; give it a default value of 0 so quiet periods still emit datapoints. A subscription filter streams the matching events themselves to Kinesis Data Streams, Firehose, or Lambda.
- **CloudTrail is audit, CloudWatch is telemetry.** "Who changed the security group" is CloudTrail; "why did CPU spike" is CloudWatch.
  - Management events are recorded by default; event history holds 90 days.
  - Data events (S3 object reads/writes, Lambda invokes) must be enabled on a trail and cost extra.
  - A trail to S3 keeps events longer; an organization trail covers every account from the management account.
  - A trail can feed CloudWatch Logs so a metric filter can alarm on API activity such as failed console logins.
- **Alarm states:** `OK`, `ALARM`, `INSUFFICIENT_DATA`. An established alarm drifting to `INSUFFICIENT_DATA` means the metric stopped arriving (stopped instance, broken agent, wrong namespace or dimensions).
- **M out of N:** period = length of each datapoint, N = evaluation periods, M = datapoints to alarm. Use 2 of 3 to stop flapping without losing detection.
- **Treat missing data:** `missing` (default), `ignore`, `breaching` (silence is failure, e.g. heartbeats), `notBreaching` (sparse error counts).
- **Actions:** SNS, an Auto Scaling policy, an EC2 action (stop, terminate, reboot, recover), or a Systems Manager OpsItem/incident. Every state change also goes to EventBridge for wider routing.
- **Composite alarms** join several child alarms using AND/OR/NOT logic to cut duplicate pages. They notify; they do not run EC2 or scaling actions.
- **"Alarm fired, nobody paged":** check for an unconfirmed SNS subscription, a wrong or empty topic, or disabled actions. The topic must be in the alarm's Region.
- **Dashboards:** each widget can pull from a different Region natively. For multiple accounts, link source accounts to a monitoring account (cross-account observability). Dashboards display; only alarms evaluate.

📖 Full lesson: [CloudWatch Metrics, Alarms, and Log Filters for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudwatch-metrics-alarms-logging/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 1.2 Identify and remediate issues by using monitoring and availability metrics

- **The loop:** detect (alarm, status check, Health event, anomaly band) → route (alarm action, SNS, EventBridge rule) → act (recover, replace, Lambda, runbook) → verify (alarm back to `OK`, runbook step success, OpsItem recorded). Answers that skip verification are weaker.
- **Isolating the fault:**
  - **Metric math** alarms on what matters, such as an error rate (`100*errors/requests`) instead of a raw count, or a fleet sum.
  - **Anomaly detection** learns a band with daily and weekly patterns. Use it for cyclical metrics; use a static threshold for absolute limits like a filling disk.
  - **CloudTrail correlation** finds change-caused incidents: CloudWatch shows when it broke, CloudTrail shows what changed. The fix is often reverting the change.
- **Status checks map to different fixes:**

| Failed check | Meaning | Fix |
|---|---|---|
| `StatusCheckFailed_System` | AWS host problem: power, network, hardware | Alarm with the **recover** action, or stop/start |
| `StatusCheckFailed_Instance` | Inside your OS: memory exhausted, corrupt filesystem, bad kernel or network config | Reboot or repair the OS. Recovery does not help |

- **Recover** keeps the instance ID, private and Elastic IPs, and EBS volumes, but loses in-memory data.
- **Auto Scaling replaces, it does not repair.** Right for stateless fleets, wrong for a stateful single instance. Add the ELB health check type so a failing app counts as unhealthy.
- **Choosing a pattern:** prefer built-in actions and predefined runbooks over custom Lambda code ("least operational overhead"). Instance problems get instance actions, fleet problems get Auto Scaling, and multi-step fixes get an Automation runbook. Keep SNS in parallel so humans know.
- **EventBridge basics:** the default bus receives AWS service events. A rule matches a JSON pattern (exact, case-sensitive, values in arrays) and sends to up to five targets. **Input transformers** reshape the payload without a Lambda. Rules can also run on schedules.
- **Rule never fires:** work this checklist.
  1. Pattern mismatch: test it against a real sample event. Alarm state lives at `state.value`.
  2. Rule disabled.
  3. Target permissions: Lambda and SNS need a resource policy, and runbook targets need an IAM role on the rule.
  4. Wrong bus.
  5. Wrong Region, because EventBridge is regional. Cross-account delivery also needs a policy on the receiving bus.
- **Systems Manager pieces:**
  - **Automation runbooks:** `AWS-` prefixed ones are predefined (for example `AWS-RestartEC2Instance`), and custom ones are written in YAML or JSON. They run under an IAM role.
  - **Run Command:** ad hoc commands at scale, with no SSH.
  - **State Manager:** reapplies configuration to fix drift.
  - **OpsCenter:** tracks OpsItems. Incident Manager adds escalation and on-call.
- **AWS Health Dashboard** shows events for your account (maintenance, retirements). Health events reach EventBridge, so scheduled retirements can trigger proactive replacement.
- **Kiro and AWS DevOps Agent** are named in the guide as AI-assisted tools for metric analysis and remediation. Know where they fit and ignore answer choices that rely on specific features.

📖 Full lesson: [Automated Remediation: EventBridge, Runbooks, and EC2 Recovery (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-automated-remediation-eventbridge-runbooks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 1.3 Implement performance optimization strategies for compute, storage, and database resources

- **Right-size from evidence.** Two weeks at 4% CPU is waste; sustained 80%+ with queuing is starvation. Memory data needs the agent.
- **Compute Optimizer** classifies EC2 as over-provisioned, under-provisioned, or optimized from CloudWatch history. It recommends specific types and also covers EBS, Lambda memory, and Auto Scaling groups. Trusted Advisor only flags idle instances. When both are options for "recommend an instance type", pick Compute Optimizer.
- **Tags** (team, app, environment, activated as cost allocation tags) let you slice dashboards and Cost Explorer and target the right fleet.
- **Families:** M/T balanced, C compute-heavy, R/X memory-heavy. An out-of-memory workload needs R, not a larger C.
- **T-family credit trap:** a service slows after sustained load or at the same busy hour daily. Look at `CPUCreditBalance` falling to zero. Unlimited mode suits occasional overruns (surplus is billed); a constantly busy box belongs on M or C.
- **EBS:**

| Type | Model | Use |
|---|---|---|
| gp3 | 3,000 IOPS and 125 MiB/s baseline at any size; IOPS and throughput raised separately from capacity | Default for boot and general use |
| gp2 | Baseline tied to size (3 IOPS/GiB); small volumes burst on credits | Legacy. Migrate to gp3 |
| io1/io2 | Provisioned IOPS SSD, consistent latency | Databases needing sustained IOPS beyond gp3 |
| st1 | Throughput HDD | Large sequential I/O. Not bootable |
| sc1 | Cold HDD, cheapest | Rarely read sequential data. Not bootable |

- **Burst exhaustion signature:** `BurstBalance` drops to 0% as `VolumeQueueLength` climbs. Fix live with **Elastic Volumes** (change gp2 to gp3 while attached, no downtime).
- **Second ceiling:** each instance type has its own aggregate EBS limit. Healthy volume metrics plus slow storage means a bigger instance, not a faster volume.
- **S3 transfer choices:**
  - **Multipart upload:** for any single large object. It is recommended above 100 MB and required above the 5 GB single-PUT cap. Add a lifecycle rule to abort incomplete uploads.
  - **Transfer Acceleration:** for distant clients, routed through edge locations. It gives no gain near the Region.
  - **DataSync:** scheduled, incremental, verified bulk copies from NFS/SMB into S3, EFS, or FSx.
  - **Prefixes:** request rates scale per prefix, so spreading hot keys adds parallel throughput.
  - **Lifecycle rules:** tier or expire aging objects.
- **Shared storage by trigger phrase:** Linux shared file system → EFS (with lifecycle to Infrequent Access for cold files). Windows share with AD → FSx for Windows File Server. HPC or ML scratch, S3-linked → FSx for Lustre. Linux and Windows on the same data (NFS, SMB, iSCSI) → FSx for NetApp ONTAP.
- **RDS diagnosis:** start with **Performance Insights**. Load above the vCPU line means sessions are waiting; top SQL and wait events name the cause, and it offers proactive recommendations. In CloudWatch, watch `CPUUtilization`, `DatabaseConnections`, `FreeableMemory`, and `ReadLatency`/`WriteLatency`.
- **RDS fixes:** connection storms (often Lambda) → **RDS Proxy**, which also softens failover. Read-heavy → read replicas (with lag). Starved resources → larger class, or gp3/provisioned IOPS storage.
- **Placement groups:** cluster for lowest node-to-node latency in one AZ (shared blast radius). Spread puts each instance on distinct hardware, capped at seven per AZ per group. Partition isolates rack groups for Hadoop, Kafka, and Cassandra and scales past spread. ENA enhanced networking complements them.

📖 Full lesson: [EC2, EBS, S3 and RDS Performance Optimization for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-ec2-ebs-rds-performance-optimization/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

[← Back to the study guide](../README.md)
