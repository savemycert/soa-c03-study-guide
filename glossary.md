# SOA-C03 Glossary

193 SOA-C03 terms and services, each in one plain-English sentence.

[A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [X](#x)

## A

- **Access advisor**: IAM data showing when a principal last used each service, used to remove unneeded permissions.
- **Account password policy**: Rules for IAM user console passwords covering length, complexity, reuse, and expiry.
- **Alias record**: A Route 53 record that points directly at an AWS resource and works at the zone apex.
- **Amazon ECR**: The AWS container registry that ECS and EKS pull images from.
- **Amazon GuardDuty**: A threat detection service that analyzes CloudTrail, VPC Flow Logs, and DNS logs.
- **Amazon Inspector**: A vulnerability scanner for EC2 instances, ECR images, and Lambda functions.
- **Amazon Macie**: A service that discovers sensitive data such as PII in S3 and flags risky bucket settings.
- **Amazon VPC IPAM**: A service for planning, tracking, and allocating IP ranges across accounts and Regions.
- **AmazonSSMManagedInstanceCore**: The AWS managed policy that gives an instance profile the permissions Systems Manager needs.
- **AMI deregistration**: Removing an AMI from use, which leaves its backing EBS snapshots in place until deleted separately.
- **Anomaly detection**: A CloudWatch model that learns a metric's normal band, including daily and weekly patterns, and alarms outside it.
- **Application Recovery Controller**: A service providing readiness checks and routing controls for managing Regional failover.
- **Aurora reader endpoint**: A single endpoint that load-balances read connections across an Aurora cluster's replicas.
- **Aurora Serverless v2**: An Aurora option that scales database capacity continuously in ACUs between a floor and a ceiling.
- **Automation runbook**: A Systems Manager document that runs a sequence of operational steps against AWS resources under an IAM role.
- **AWS CDK**: A framework for defining infrastructure in programming languages that synthesizes CloudFormation templates.
- **AWS Client VPN**: A managed remote-access VPN for individual users connecting to a VPC.
- **AWS Compute Optimizer**: A service that analyzes utilization history and recommends better-sized EC2, EBS, Lambda, and Auto Scaling configurations.
- **AWS Config rule**: A check that marks resources compliant or noncompliant against a desired configuration.
- **AWS Control Tower**: A service that sets up a governed multi-account landing zone with preconfigured guardrails.
- **AWS DataSync**: A managed service for scheduled, incremental, verified bulk transfers into S3, EFS, or FSx.
- **AWS Global Accelerator**: A service that provides two static anycast IPs and routes TCP and UDP traffic to healthy regional endpoints.
- **AWS Health Dashboard**: The account-specific view of AWS events affecting your resources, including scheduled maintenance and retirements.
- **AWS Network Firewall**: A managed stateful firewall deployed in firewall subnets and reached through route table entries.
- **AWS RAM**: The service that shares resources such as subnets and Transit Gateways with other accounts.
- **AWS Secrets Manager**: A secrets store with automatic rotation, including a managed integration for RDS.
- **AWS Security Hub**: A service that aggregates security findings and scores accounts against security standards.
- **AWS Service Catalog**: A service for publishing approved CloudFormation-based products that users can launch themselves.
- **AWS Step Functions**: A service that orchestrates multi-step workflows with branching and retries.
- **AWS-RunPatchBaseline**: The Systems Manager document that scans for or installs patches according to the baseline.
- **aws:MultiFactorAuthPresent**: The condition key used in a deny policy to require MFA before other actions are allowed.
- **aws:PrincipalOrgID**: A condition key that allows any principal from a given AWS Organizations organization.
- **aws:RequestedRegion**: A condition key that matches the Region an API call targets, used for Region restrictions.
- **aws:SecureTransport**: A condition key that is false for plain HTTP requests, used to enforce TLS.
- **aws:SourceIp**: A condition key that limits requests to specified public IP ranges.

## B

- **Backup plan**: An AWS Backup policy of rules that set backup schedules, retention, and cold-storage lifecycle.
- **Backup vault**: An encrypted AWS Backup container that stores recovery points under its own access policy.
- **Blue/green deployment**: A release method that builds a parallel environment and shifts traffic to it once validated.
- **BurstBalance**: The EBS metric showing the percentage of burst credits left on gp2, st1, and sc1 volumes.

## C

- **Cache policy**: The CloudFront setting that defines the cache key and TTL limits for a behavior.
- **Calculated health check**: A Route 53 health check whose status depends on how many of its child checks are healthy.
- **Change set**: A preview of the additions, modifications, removals, and replacements a stack update would make.
- **CIDR block**: The IP address range assigned to a VPC or subnet.
- **CloudWatch agent**: Software installed in the guest OS that publishes memory, disk-space, and other in-guest metrics plus log files.
- **CloudWatch Network Monitor**: A service that probes hybrid network paths and reports packet loss and latency.
- **Cluster placement group**: A placement strategy that packs instances close together in one AZ for the lowest inter-node latency.
- **Composite alarm**: An alarm built from other alarms with AND, OR, and NOT logic to reduce duplicate notifications.
- **Conditions**: A CloudFormation template section that decides whether resources are created, based on parameter values.
- **Conformance pack**: A deployable bundle of Config rules and remediation actions for accounts and Regions.
- **Container Insights**: A CloudWatch feature that collects cluster, service, task, and pod metrics and logs from ECS and EKS.
- **Cooldown**: The wait after a simple scaling activity before that policy may act again.
- **CPU credits**: The balance a burstable T-family instance earns below baseline and spends to run above it.
- **Credential report**: A downloadable list of IAM users with password age, access key age, and MFA status.
- **Cross-account observability**: A CloudWatch setup that links source accounts to a monitoring account for shared metrics, logs, and dashboards.
- **Cross-zone load balancing**: A load balancer setting that lets nodes in one AZ send traffic to targets in other AZs.
- **Custom action**: A Security Hub feature that sends selected findings to EventBridge for a defined response.
- **Customer gateway**: The AWS resource that represents your on-premises VPN device.

## D

- **Data classification**: A scheme that assigns sensitivity tiers to data so each tier gets matching controls.
- **Data event**: A high-volume CloudTrail data-plane record, such as an S3 object read, that must be enabled on a trail.
- **Data Lifecycle Manager**: A service that creates, retains, and deletes EBS snapshots on a tag-targeted schedule.
- **Datapoints to alarm**: The M in an "M out of N" alarm setting, the count of breaching datapoints needed to enter ALARM.
- **DAX**: An in-memory read cache for DynamoDB that requires no application-managed cache logic.
- **Default EBS encryption**: A per-Region account setting that encrypts all newly created EBS volumes and snapshot copies.
- **Delete marker**: The placeholder S3 writes as the current version when an object in a versioned bucket is deleted.
- **DeletionPolicy**: A resource attribute that deletes, retains, or snapshots the resource when it leaves the stack.
- **Deregistration delay**: The time a draining target keeps serving in-flight requests after it stops receiving new ones.
- **Desired capacity**: The instance count an Auto Scaling group works to maintain, kept between its minimum and maximum.
- **Detailed monitoring**: A paid EC2 option that publishes the standard instance metrics every minute instead of every five minutes.
- **Dimension**: A name-value pair that scopes a metric to a specific resource, making each combination a distinct metric.
- **DNS validation**: The ACM method that proves domain control with a CNAME record and enables automatic renewal.
- **Drift detection**: A CloudFormation check that reports resources whose live configuration no longer matches the template.

## E

- **EC2 Image Builder**: A managed pipeline service that builds, tests, and distributes AMIs and container images on a schedule.
- **ECR lifecycle policy**: A repository rule that expires old or untagged container images automatically.
- **Egress-only internet gateway**: A gateway that allows outbound IPv6 connections while blocking inbound-initiated ones.
- **Elastic Volumes**: The EBS capability to change a volume's type, size, and performance while it stays attached.
- **Endpoint policy**: A resource policy on a VPC endpoint that limits what can be done through it.
- **Endpoint service**: A service published behind a Network Load Balancer so other VPCs can reach it through PrivateLink.
- **Envelope encryption**: Encrypting data with a data key and protecting that data key with a KMS key.
- **Ephemeral ports**: The high-numbered client ports that return traffic uses, which stateless NACLs must allow.
- **ErrorPortAllocation**: A NAT gateway metric that counts failed source-port allocations, indicating port exhaustion.
- **Evaluate Target Health**: An alias record setting that inherits the health of the AWS resource the alias points to.
- **Explicit deny**: A policy statement that blocks an action regardless of any allow elsewhere.
- **Export**: A named CloudFormation output that other stacks can read with `Fn::ImportValue`.

## F

- **Failover routing**: A Route 53 policy that answers with a secondary record only while the primary record is unhealthy.
- **FSx for Lustre**: A high-throughput parallel file system for HPC and machine learning that can link to an S3 bucket.

## G

- **Gateway endpoint**: A free route-table-based VPC endpoint for S3 and DynamoDB.
- **Geoproximity routing**: A Route 53 policy that routes by distance to resources, adjustable with a bias value.
- **Golden image**: A pre-hardened, versioned machine or container image used as the standard base for launches.

## H

- **Health check grace period**: The time an Auto Scaling group waits after launch before it evaluates a new instance's health.
- **High-resolution metric**: A custom metric stored at down to one-second granularity, supporting alarms on 10- or 30-second periods.
- **Hot partition**: A DynamoDB partition receiving a disproportionate share of traffic, causing throttling despite spare table capacity.

## I

- **IAM Access Analyzer**: A service that reports resources reachable from outside your account or organization and unused access.
- **IAM Identity Center**: The workforce sign-on service that assigns permission sets to users across an organization's accounts.
- **IAM policy simulator**: A tool that evaluates whether a principal would be allowed an action, without performing it.
- **Identity-based policy**: A policy attached to a user, group, or role that states what that principal may do.
- **Imported certificate**: A certificate brought into ACM from elsewhere, which ACM never renews.
- **Inbound Resolver endpoint**: ENIs that let on-premises DNS servers forward queries into a VPC's Resolver.
- **Input transformer**: An EventBridge feature that extracts fields from an event and reshapes the payload before delivery.
- **Instance profile**: The container that delivers an IAM role's temporary credentials to an EC2 instance.
- **Instance refresh**: An Auto Scaling feature that replaces instances in rolling fashion while keeping a minimum healthy percentage.
- **Instance warmup**: The period during which a new instance's metrics are excluded from the group aggregate used for scaling.
- **INSUFFICIENT_DATA**: The alarm state meaning there are not enough datapoints to evaluate, often because the metric stopped arriving.
- **Interface endpoint**: A PrivateLink ENI in your subnet that gives private access to an AWS or partner service.
- **Internet gateway**: The VPC component that connects subnets with public IPs to the internet.
- **Invalidation**: A CloudFront request that removes cached objects from edge locations before they expire.

## K

- **Key policy**: The resource-based policy on a KMS key that is the primary authority over its use.
- **KMS grant**: A programmatic permission to use a KMS key, often created by AWS services.

## L

- **Launch template**: A versioned definition of the AMI, instance type, networking, and user data an Auto Scaling group launches.
- **Lifecycle hook**: A pause in an instance's launch or termination so automation can configure or drain it.
- **Local route**: The permanent route table entry that keeps traffic within the VPC CIDR routable.
- **Log group**: The CloudWatch Logs unit where retention, access, and filters are configured for a set of log streams.
- **Logical ID**: The name a resource has inside a CloudFormation template, used for references and stack policies.
- **Logs Insights**: The interactive query engine for searching and aggregating log data that has already been ingested.

## M

- **Maintenance window**: A scheduled period with a duration and cutoff during which disruptive tasks may run.
- **Management account**: The account that owns an organization and is not restricted by SCPs.
- **Mappings**: A CloudFormation template section of static lookup tables, such as AMI IDs per Region.
- **Metric filter**: A log group rule that increments a CloudWatch metric each time a pattern matches new log events.
- **Metric math**: Expressions computed over one or more metrics, such as an error rate, that alarms and graphs can use.
- **Multi-AZ DB cluster**: An RDS deployment with one writer and two readable standbys in separate AZs.
- **Multipart upload**: An S3 method that uploads a large object as parallel parts that S3 reassembles.
- **Multivalue answer routing**: A Route 53 policy that returns several healthy records for the client to choose from.

## N

- **Namespace**: A container that groups CloudWatch metrics, such as `AWS/EC2` or a custom application namespace.
- **NAT gateway**: A managed zonal service in a public subnet that gives private subnets outbound-only IPv4 internet access.
- **Nested stack**: A child stack created by a parent template through the `AWS::CloudFormation::Stack` resource.
- **Network ACL**: A stateless subnet-level filter with numbered allow and deny rules.

## O

- **OpsItem**: A Systems Manager OpsCenter work item that records an operational issue with related resources and runbooks.
- **Organization trail**: A CloudTrail trail created in the management account that logs activity for every account in the organization.
- **Origin Access Control (OAC)**: The CloudFront feature that signs requests to a private S3 origin so only the distribution can read it.
- **Origin request policy**: The CloudFront setting that forwards values to the origin without adding them to the cache key.
- **Outbound Resolver endpoint**: ENIs that let the Resolver forward matching queries to on-premises DNS servers through forwarding rules.

## P

- **Partition placement group**: A placement strategy that isolates groups of instances on racks that share no hardware with other groups.
- **Patch baseline**: The Patch Manager rules defining which patches are approved, including an auto-approval delay.
- **Patch group**: A tag-based mapping of instances to a specific patch baseline.
- **Performance Insights**: An RDS feature that charts database load by wait event, SQL statement, host, and user.
- **Permission set**: An Identity Center bundle of policies provisioned as a role in each assigned account.
- **Permissions boundary**: A managed policy that caps the maximum permissions of one user or role without granting any.
- **Pilot light**: A DR pattern that keeps replicated data live in the recovery Region while the application tier stays off.
- **Point-in-time recovery (PITR)**: Continuous backup that restores a database or table to a chosen moment within a retention window.
- **Predictive scaling**: A scaling policy that forecasts load from history and adds capacity before the expected demand.
- **Private hosted zone**: A Route 53 zone that answers queries only from the VPCs associated with it.
- **Provisioned concurrency**: A Lambda setting that keeps initialized execution environments ready for latency-sensitive functions.

## R

- **RDS Proxy**: A managed connection pool between applications and an RDS database that absorbs connection storms.
- **Recover action**: An alarm action that moves an instance with a failed system status check onto healthy hardware.
- **Recovery point objective (RPO)**: The maximum acceptable data loss, measured as time back to the last usable copy.
- **Recovery time objective (RTO)**: The maximum acceptable time from failure to restored service.
- **Replacement**: A CloudFormation update that deletes a resource and creates a new one because an immutable property changed.
- **ReplicaLag**: The metric showing how far an asynchronous RDS read replica is behind its primary.
- **Reserved concurrency**: A Lambda setting that caps a function's concurrency and guarantees it that share of the account pool.
- **Resolver query logging**: Logging of DNS queries made from within VPCs to CloudWatch Logs, S3, or Firehose.
- **Resource-based policy**: A policy attached to a resource that names which principals may access it.
- **ROLLBACK_COMPLETE**: The terminal state of a stack whose creation failed and was rolled back, requiring deletion.
- **Route 53 Resolver**: The built-in VPC DNS server at the VPC base address plus two.
- **Route 53 Resolver DNS Firewall**: A filter that blocks or allows outbound DNS queries from VPCs based on domain lists.
- **Route table**: A set of routes that determines where traffic from its associated subnets is sent.
- **Run Command**: A Systems Manager capability that runs a document across many instances once, with rate controls.

## S

- **S3 Batch Replication**: A feature that replicates objects that existed before a replication rule was created.
- **S3 Event Notifications**: Bucket-level events for object creation and removal delivered to Lambda, SQS, or SNS.
- **S3 Transfer Acceleration**: An S3 feature that routes uploads from distant clients through edge locations onto the AWS network.
- **SAML 2.0 federation**: A per-account trust with an identity provider that lets users assume roles through SAML assertions.
- **Scheduled scaling**: A scaling action that changes group capacity at set times or on a recurring schedule.
- **SecureString parameter**: A KMS-encrypted Parameter Store value suitable for static secrets.
- **Service control policy (SCP)**: An Organizations policy that caps permissions in member accounts and never grants access.
- **Service role**: An IAM role attached to a stack that CloudFormation uses instead of the deployer's credentials.
- **Service-managed permissions**: A StackSets mode that uses AWS Organizations to create roles and auto-deploy to new accounts.
- **Session Manager**: A Systems Manager capability for IAM-controlled, logged shell access with no SSH keys or inbound ports.
- **SSE-KMS**: S3 server-side encryption with a KMS key, giving key-level control and CloudTrail records of key use.
- **SSM Agent**: The agent that lets Systems Manager manage an instance over an outbound connection.
- **Stack policy**: A JSON document that blocks update actions on specified resources during stack updates.
- **StackSets**: A CloudFormation feature that deploys one template as stacks across many accounts and Regions.
- **State Manager association**: A binding of a document to targets on a schedule that keeps reapplying the desired configuration.
- **Static stability**: The ability to survive an AZ failure without launching or changing anything at the moment of failure.
- **Step scaling policy**: A scaling policy whose adjustment size depends on how far a metric has crossed its alarm threshold.
- **Storage autoscaling**: An RDS feature that grows allocated storage automatically toward a set maximum when free space runs low.
- **Subscription filter**: A log group rule that streams matching log events to Kinesis Data Streams, Firehose, or Lambda.
- **Suspended process**: An Auto Scaling process, such as Launch or HealthCheck, that has been stopped manually or by the group.
- **Systems Manager Inventory**: A capability that collects installed software, versions, and OS details from managed instances.

## T

- **Target group**: A set of load balancer targets that share one health check configuration.
- **Target tracking policy**: A scaling policy that holds a chosen metric at a target value using alarms AWS manages.
- **Termination protection**: A stack setting that prevents anyone from deleting the entire stack while it is enabled.
- **Terraform state file**: The file where Terraform records the resources it manages, which you must store and lock.
- **Transit Gateway**: A regional hub that connects many VPCs and VPNs with transitive, segmentable routing.
- **Treat missing data**: The alarm setting that decides whether gaps in a metric count as missing, ignored, breaching, or not breaching.
- **Trusted Advisor**: A service that checks accounts against best practices, including security checks such as open ports and root MFA.
- **TTL**: The time a resolver may cache a DNS answer before asking again.

## U

- **Unlimited mode**: A T-family setting that allows bursting past a zero credit balance, with surplus usage billed.
- **UPDATE_ROLLBACK_FAILED**: The state of a stack whose update rollback could not finish, recovered with continue update rollback.

## V

- **Vault Lock**: An AWS Backup setting that prevents recovery points from being deleted before their retention ends.
- **Virtual private gateway**: The AWS-side VPN endpoint attached to a single VPC.
- **VPC Flow Logs**: Records of IP traffic metadata captured for a VPC, subnet, or network interface.
- **VPC peering**: A non-transitive one-to-one private connection between two VPCs with non-overlapping CIDRs.
- **VPC Reachability Analyzer**: A tool that analyzes configuration between two endpoints and names the component blocking the path.

## W

- **Warm standby**: A DR pattern that runs a scaled-down but complete copy of the stack in the recovery Region.

## X

- **x-edge-result-type**: A CloudFront log field showing whether a request was a hit, a miss, or an error.

[← Back to the study guide](README.md)
