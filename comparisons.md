# SOA-C03 Commonly Confused Services and Concepts

Side-by-side comparisons of the SOA-C03 services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization](#domain-1-monitoring-logging-analysis-remediation-and-performance-optimization)
- [Domain 2: Reliability and Business Continuity](#domain-2-reliability-and-business-continuity)
- [Domain 3: Deployment, Provisioning, and Automation](#domain-3-deployment-provisioning-and-automation)
- [Domain 4: Security and Compliance](#domain-4-security-and-compliance)
- [Domain 5: Networking and Content Delivery](#domain-5-networking-and-content-delivery)

## Domain 1: Monitoring, Logging, Analysis, Remediation, and Performance Optimization

### CloudWatch vs CloudTrail

|                  | Amazon CloudWatch                               | AWS CloudTrail                                                      |
| ---------------- | ----------------------------------------------- | ------------------------------------------------------------------- |
| Records          | Metrics, logs, alarms, dashboards               | API calls: who, what, when, from where, result                      |
| Answers          | How is the system performing?                   | Who changed this resource?                                          |
| Default coverage | Service metrics; agent needed for in-guest data | Management events kept 90 days in event history; data events opt-in |
| Longer retention | Log group retention setting                     | A trail delivering to S3                                            |
| Exam cue         | "CPU spike", "error rate", "threshold"          | "which user modified", "who deleted"                                |

**Remember:** CloudTrail is the record and CloudWatch is the reaction. Send a trail to CloudWatch Logs when you need to alarm on specific API calls.

### Metric Filters vs Subscription Filters

|             | Metric filter                                | Subscription filter                          |
| ----------- | -------------------------------------------- | -------------------------------------------- |
| Output      | A CloudWatch metric counting pattern matches | The matching log events themselves           |
| Destination | CloudWatch metrics, then an alarm            | Kinesis Data Streams, Firehose, or Lambda    |
| History     | Only data ingested after creation            | Streams in near real time                    |
| Use it for  | Alerting on a log pattern                    | Central analysis, transformation, export     |
| Exam cue    | "alert when ERROR appears in the logs"       | "stream logs to another system in real time" |

**Remember:** a metric filter produces a number and a subscription filter moves data. Give metric filters a default value of 0 so the alarm sees quiet periods.

### System Status Check vs Instance Status Check

|                     | `StatusCheckFailed_System`                        | `StatusCheckFailed_Instance`                        |
| ------------------- | ------------------------------------------------- | --------------------------------------------------- |
| Layer               | AWS host: power, network, hardware                | Your OS: memory, filesystem, kernel, network config |
| Owner of the fault  | AWS                                               | You                                                 |
| Fix                 | Alarm with the recover action, or stop/start      | Reboot or repair the OS                             |
| What recovery keeps | Instance ID, private and Elastic IPs, EBS volumes | Not applicable; recovery does not help              |
| Exam cue            | "underlying hardware failure"                     | "corrupted filesystem", "kernel panic"              |

**Remember:** new hardware cannot fix a broken OS. Recover only answers the system check.

### gp2 vs gp3

|                   | gp2                                                       | gp3                                                 |
| ----------------- | --------------------------------------------------------- | --------------------------------------------------- |
| Performance model | Baseline scales with size; small volumes burst on credits | Fixed 3,000 IOPS and 125 MiB/s baseline at any size |
| More performance  | Grow the volume                                           | Provision IOPS and throughput separately            |
| Failure signature | `BurstBalance` at 0%, `VolumeQueueLength` rising          | No credit mechanism to run out                      |
| Change method     | Elastic Volumes, live and attached                        | Elastic Volumes, live and attached                  |
| Exam cue          | "slows after 20 minutes every night"                      | "raise IOPS without adding capacity"                |

**Remember:** a burst-exhausted gp2 volume is fixed by migrating it to gp3 in place, with no downtime. If volume metrics look clean, check the instance's EBS ceiling.

### Compute Optimizer vs Trusted Advisor

|          | AWS Compute Optimizer                                                   | AWS Trusted Advisor                                  |
| -------- | ----------------------------------------------------------------------- | ---------------------------------------------------- |
| Method   | Machine learning over CloudWatch utilization history                    | Best-practice checks across cost, security, and more |
| Output   | Over-, under-provisioned, or optimized, with specific replacement types | Flags such as idle or low-utilization instances      |
| Coverage | EC2, EBS, Lambda memory, Auto Scaling groups                            | Broad account checks                                 |
| Exam cue | "recommend an instance type from historical utilization"                | "identify idle resources"                            |

**Remember:** Trusted Advisor points at waste. Compute Optimizer tells you what to replace it with.

## Domain 2: Reliability and Business Continuity

### Target Tracking vs Step Scaling

|               | Target tracking                      | Step scaling                                     |
| ------------- | ------------------------------------ | ------------------------------------------------ |
| You configure | A metric and a target value          | Your own CloudWatch alarms plus adjustment steps |
| Alarms        | Created and managed by AWS           | Created and managed by you                       |
| Response      | Scales out and in to hold the target | Adjustment sized by how far the metric breached  |
| Flapping risk | Low; scales in conservatively        | Higher when out/in thresholds sit close together |
| Exam cue      | "keep average CPU near 50%"          | "add more instances the worse the breach"        |

**Remember:** target tracking is the default answer. Simple scaling is legacy, and a shorter cooldown never fixes flapping.

### Application Load Balancer vs Network Load Balancer

|              | ALB                                          | NLB                                        |
| ------------ | -------------------------------------------- | ------------------------------------------ |
| Layer        | 7: HTTP and HTTPS                            | 4: TCP, UDP, TLS                           |
| Routing      | Path, host, header, query string             | Listener port to target group              |
| IP addresses | Changing; use the DNS name                   | Static per AZ, Elastic IP support          |
| Client IP    | `X-Forwarded-For` header                     | Preserved at the connection                |
| Cross-zone   | On by default                                | Off by default                             |
| Exam cue     | "route /api and /images to different fleets" | "static IP", "UDP", "allowlist a fixed IP" |

**Remember:** an NLB can front an ALB when an HTTP app needs both a fixed IP and layer-7 routing.

### RDS Multi-AZ Standby vs Read Replica

|                           | Multi-AZ instance standby             | Read replica                      |
| ------------------------- | ------------------------------------- | --------------------------------- |
| Purpose                   | Availability                          | Read scaling                      |
| Replication               | Synchronous                           | Asynchronous (watch `ReplicaLag`) |
| Serves reads              | Never                                 | Yes                               |
| Failover                  | Automatic; same endpoint              | Manual promotion by an operator   |
| Protects against bad data | No; corruption replicates             | No; it replicates too             |
| Exam cue                  | "survive an AZ failure automatically" | "offload reporting queries"       |

**Remember:** if a stem needs automatic failover and readable standbys, the answer is a Multi-AZ DB cluster. Neither option is a backup.

### Pilot Light vs Warm Standby

|                         | Pilot light                                           | Warm standby                            |
| ----------------------- | ----------------------------------------------------- | --------------------------------------- |
| Running before disaster | Replicated data layer only                            | Whole stack, scaled down                |
| Work at failover        | Start and scale the app tier, switch traffic          | Scale up and switch traffic             |
| RTO                     | Tens of minutes                                       | Minutes                                 |
| Standing cost           | Low                                                   | High                                    |
| Exam cue                | "data must be current; servers can start on failover" | "a scaled-down copy is already running" |

**Remember:** both keep data current. Pick the cheaper pattern that still meets the stated RTO and RPO.

### ElastiCache for Redis vs Memcached

|                          | Redis                                | Memcached                                |
| ------------------------ | ------------------------------------ | ---------------------------------------- |
| Replication and failover | Yes, automatic with Multi-AZ         | No                                       |
| Persistence              | Options available                    | None                                     |
| Scaling                  | Replicas; cluster mode shards writes | Add nodes horizontally; multi-threaded   |
| Losing a node            | Replica takes over                   | Cold misses fall through to the database |
| Exam cue                 | "cache must survive node failure"    | "simple ephemeral object cache"          |

**Remember:** when losing the cache would crush the database, choose Redis.

## Domain 3: Deployment, Provisioning, and Automation

### CloudFormation StackSets vs AWS RAM

|                 | StackSets                                                  | AWS Resource Access Manager                      |
| --------------- | ---------------------------------------------------------- | ------------------------------------------------ |
| Model           | Copies a template's resources into each account and Region | Shares one existing resource with other accounts |
| Ownership       | Each account gets its own stack                            | Owner keeps control; consumers use it in place   |
| Typical targets | Baseline roles, Config rules, logging                      | Subnets, Transit Gateway, Resolver rules         |
| Org integration | Service-managed permissions auto-deploy to new OU accounts | Share with the organization or specific accounts |
| Exam cue        | "every account must get this stack"                        | "many accounts should use one central VPC"       |

**Remember:** StackSets replicate and RAM shares. Service Catalog is the third option when users should launch only approved products.

### Change Sets vs Drift Detection

|                  | Change set                                    | Drift detection                                   |
| ---------------- | --------------------------------------------- | ------------------------------------------------- |
| Compares         | A proposed template against the running stack | Live resource configuration against the template  |
| When             | Before an update                              | Any time after deployment                         |
| Output           | Add, Modify, Remove, and Replacement flags    | `IN_SYNC`, `MODIFIED`, `DELETED`; stack `DRIFTED` |
| Changes anything | Only when you execute it                      | Never; it only reports                            |
| Exam cue         | "preview changes before updating"             | "someone edited the resource in the console"      |

**Remember:** a change set looks forward at your edit, and drift detection looks back at someone else's.

### Stack Policy vs DeletionPolicy

|                | Stack policy                                       | DeletionPolicy attribute                |
| -------------- | -------------------------------------------------- | --------------------------------------- |
| Scope          | Whole stack, naming protected logical IDs          | One resource                            |
| Guards against | Updates that would modify or replace a resource    | Deletion of the resource or the stack   |
| Options        | Deny or allow update actions; override when needed | `Delete`, `Retain`, `Snapshot`          |
| Exam cue       | "prevent updates from replacing the database"      | "keep the data if the stack is deleted" |

**Remember:** protect a production database with both, and add termination protection to guard the whole stack.

### Run Command vs State Manager vs Automation Runbooks

|          | Run Command                                   | State Manager                                | Automation runbook                             |
| -------- | --------------------------------------------- | -------------------------------------------- | ---------------------------------------------- |
| Shape    | One document on many instances                | Association: document + targets + schedule   | Multi-step workflow with API calls and scripts |
| Cadence  | One-off                                       | Continuous; reapplies to fix drift           | On demand, scheduled, or event-driven          |
| Extras   | Tag targeting, rate controls, S3 output       | Covers instances launched later              | Approval steps, rate controls                  |
| Exam cue | "restart the service on every web server now" | "keep the agent installed on every instance" | "standardize a multi-step procedure"           |

**Remember:** read the verb. Once is Run Command, always is State Manager, and step by step is a runbook.

### CloudFormation vs Terraform

|          | CloudFormation (and the CDK)     | Terraform                                |
| -------- | -------------------------------- | ---------------------------------------- |
| State    | Held by AWS in the stack         | A state file you store, secure, and lock |
| Scope    | AWS                              | Multi-cloud                              |
| Preview  | Change set                       | Plan                                     |
| Drift    | On-demand drift detection        | Refresh during plan                      |
| Exam cue | "AWS-native, no state to manage" | "one tool across clouds"                 |

**Remember:** the CDK is not a separate engine. It synthesizes CloudFormation, so every stack behavior applies to it.

## Domain 4: Security and Compliance

### SCPs vs IAM Policies vs Permissions Boundaries

|                    | Service control policy                      | IAM identity policy     | Permissions boundary                        |
| ------------------ | ------------------------------------------- | ----------------------- | ------------------------------------------- |
| Attaches to        | Organization root, OU, or account           | User, group, or role    | One user or role                            |
| Grants access      | No, caps the maximum                        | Yes                     | No, caps the maximum                        |
| Management account | Not affected                                | Applies                 | Applies                                     |
| Use it for         | Org-wide guardrails such as blocked Regions | Day-to-day permissions  | Letting teams create roles safely           |
| Exam cue           | "no one in any account may"                 | "grant the team access" | "delegate role creation without escalation" |

**Remember:** only identity and resource policies grant. "The SCP allows it but the user is denied" means no IAM policy grants the action.

### CloudTrail vs IAM Policy Simulator vs IAM Access Analyzer

|          | CloudTrail                                   | Policy simulator                              | Access Analyzer                            |
| -------- | -------------------------------------------- | --------------------------------------------- | ------------------------------------------ |
| Question | What happened?                               | What would happen?                            | What could outsiders reach?                |
| Input    | Recorded API calls                           | A principal, action, and resource             | Resource-based policies                    |
| Output   | `userIdentity`, `eventName`, source IP, time | Allowed or denied, and the deciding statement | External access and unused-access findings |
| Exam cue | "who deleted the bucket"                     | "test a policy without running it"            | "bucket shared outside the organization"   |

**Remember:** CloudTrail is the past, the simulator is a hypothetical, and Access Analyzer is exposure.

### Secrets Manager vs Parameter Store SecureString

|            | AWS Secrets Manager                         | Parameter Store SecureString            |
| ---------- | ------------------------------------------- | --------------------------------------- |
| Built for  | Secrets                                     | Configuration, including static secrets |
| Encryption | KMS                                         | KMS                                     |
| Rotation   | Automatic, managed for RDS                  | None built in                           |
| Cost       | Per secret per month                        | Standard tier free                      |
| Exam cue   | "rotate database credentials automatically" | "cheapest encrypted storage for config" |

**Remember:** both are fetched at run time by the workload's role. Rotation decides the choice.

### GuardDuty vs Inspector vs Macie

|                 | Amazon GuardDuty                           | Amazon Inspector                               | Amazon Macie                               |
| --------------- | ------------------------------------------ | ---------------------------------------------- | ------------------------------------------ |
| Finds           | Active threats and abuse                   | Known CVEs and unintended network reachability | Sensitive data such as PII and credentials |
| Looks at        | CloudTrail, VPC Flow Logs, DNS logs        | EC2, ECR images, Lambda                        | S3 objects and bucket posture              |
| Blocks anything | No                                         | No                                             | No                                         |
| Exam cue        | "instance talking to a crypto-mining pool" | "unpatched vulnerabilities on instances"       | "PII sitting in S3"                        |

**Remember:** Inspector reports what could be exploited and GuardDuty what is being exploited. Security Hub gathers all three into one view.

### AWS Config vs AWS Security Hub

|             | AWS Config                                    | AWS Security Hub                                    |
| ----------- | --------------------------------------------- | --------------------------------------------------- |
| Role        | Records configuration and evaluates rules     | Aggregates findings and scores security standards   |
| Sources     | Its own recorder                              | GuardDuty, Inspector, Macie, Config, partners       |
| Remediation | Rules attach SSM runbooks directly            | Custom actions and EventBridge rules                |
| Scale       | Conformance packs across accounts and Regions | Delegated administrator across accounts and Regions |
| Exam cue    | "resources must match policy and auto-fix"    | "single pane of glass", "security score"            |

**Remember:** pick the specialist when the stem describes one symptom, and Security Hub when it asks for a consolidated view.

## Domain 5: Networking and Content Delivery

### Security Groups vs Network ACLs

|                    | Security group                             | Network ACL                                    |
| ------------------ | ------------------------------------------ | ---------------------------------------------- |
| Attaches to        | Network interface                          | Subnet                                         |
| State              | Stateful: replies allowed automatically    | Stateless: replies need their own rules        |
| Rules              | Allow only, all evaluated together         | Allow and deny, lowest rule number first       |
| Flow log signature | Can reject a request, never only its reply | ACCEPT request then REJECT reply on one ENI    |
| Exam cue           | "only the app tier may reach the database" | "block a hostile CIDR", "replies never return" |

**Remember:** when a request is accepted but its reply is rejected, look for a NACL without ephemeral-port rules.

### Gateway Endpoint vs Interface Endpoint

|                | Gateway endpoint                          | Interface endpoint                                        |
| -------------- | ----------------------------------------- | --------------------------------------------------------- |
| Services       | S3 and DynamoDB only                      | Most AWS services, plus PrivateLink services              |
| Mechanism      | Route-table entry (managed prefix list)   | ENIs with private IPs in your subnets                     |
| Cost           | None                                      | Hourly per AZ plus per GB                                 |
| Common failure | Subnet's route table not associated       | Private DNS off, or security group missing 443            |
| Exam cue       | "private S3 access at no additional cost" | "private access to Systems Manager from a private subnet" |

**Remember:** gateway endpoints cut NAT processing charges for S3 and DynamoDB. Interface endpoints are chosen for privacy and carry their own hourly and per-GB charges.

### VPC Peering vs Transit Gateway

|              | VPC peering                | Transit Gateway                                  |
| ------------ | -------------------------- | ------------------------------------------------ |
| Topology     | One-to-one                 | Hub and spoke                                    |
| Transitive   | Never                      | Yes, by default                                  |
| Routing work | Routes on both VPCs        | VPC route tables plus TGW route tables           |
| Cost posture | No hourly charge           | Hourly per attachment plus per GB                |
| Exam cue     | "connect two VPCs cheaply" | "dozens of VPCs and a VPN", "transitive routing" |

**Remember:** if A peers with B and B with C, A still cannot reach C. That need is Transit Gateway.

### Inbound vs Outbound Resolver Endpoints

|                           | Inbound endpoint                                             | Outbound endpoint                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------- |
| Direction                 | On premises asks AWS                                         | VPC asks on premises                  |
| Configured where          | On-prem DNS conditionally forwards to the endpoint IPs       | Forwarding rules associated with VPCs |
| Instances change anything | No                                                           | No; they keep using the .2 resolver   |
| Exam cue                  | "data-center servers must resolve private hosted zone names" | "EC2 must resolve corp.example.com"   |

**Remember:** name the direction of the query first. Full hybrid DNS uses both endpoints over an existing private link.

### CloudFront vs Global Accelerator

|           | Amazon CloudFront                     | AWS Global Accelerator                                                 |
| --------- | ------------------------------------- | ---------------------------------------------------------------------- |
| Protocols | HTTP and HTTPS                        | TCP and UDP                                                            |
| Caching   | Yes, at edge locations                | None                                                                   |
| Addresses | Distribution domain name              | Two static anycast IPs                                                 |
| Failover  | Origin failover within a distribution | Health-based shift between Regions, no DNS wait                        |
| Exam cue  | "cache content at the edge"           | "static IPs for allowlists", "UDP gaming", "instant Regional failover" |

**Remember:** cache and content mean CloudFront. Static IP and non-HTTP mean Global Accelerator.

[← Back to the study guide](README.md)
