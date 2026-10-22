# Domain 2: Reliability and Business Continuity (22%)

This domain tests whether a workload grows and shrinks on its own, survives instance, AZ, and Region failures, and can be recovered from backups within stated RTO and RPO targets. Questions describe a misbehaving fleet, a failover that did not happen, or a recovery objective, and ask for the one control that fits.

## 2.1 Implement scalability and elasticity

- **Auto Scaling group basics:** policies move `desired` between `min` and `max`, and the group launches or terminates instances to match. Use a versioned **launch template** (roll forward or back by version). Launch configurations are legacy.
- **Health:** EC2 status checks are the default. Add the **ELB** health check type so a broken app on a healthy VM is replaced. The **grace period** delays health evaluation for new instances. If it is too short, instances are killed while booting in a loop.
- **AZ rebalancing** keeps instances even across zones. **Scale-in protection** shields instances doing long jobs.

| Policy          | Behavior                                                     | Choose when                             |
| --------------- | ------------------------------------------------------------ | --------------------------------------- |
| Target tracking | Holds a metric at a target; AWS manages the alarms both ways | "Keep CPU near 50%". The default answer |
| Step            | Adjustment size depends on breach size                       | Different responses per severity        |
| Simple          | One adjustment, then waits out the cooldown                  | Almost never. Legacy and slow           |
| Scheduled       | Sets capacity at set times                                   | "Every weekday at 9 a.m."               |
| Predictive      | Forecasts from history and launches ahead                    | Recurring patterns with long boot times |

- Policies can be combined (target tracking plus scheduled is common). When several fire, the one asking for the most capacity wins.
- **Cooldown vs warmup vs grace:**
  - The cooldown (default 300 s) applies to simple scaling.
  - Warmup keeps new instances out of the aggregate metric (target tracking and step).
  - Grace delays health checks.
- **Lifecycle hooks** pause at `Pending:Wait` (configure, prewarm) or `Terminating:Wait` (drain, ship logs). Instances stuck before service often point to a hook nobody completed.
- **Troubleshooting:**
  - **Flapping:** close out/in thresholds plus booting instances dragging the average down. Fix with one target tracking policy, a longer warmup, or wider thresholds. A shorter cooldown is the distractor.
  - **Stuck at max:** `desired` already equals `max`, or launches are failing (quota, instance type unavailable in an AZ, deleted AMI). Read the activity history first.
  - **Doing nothing:** a suspended `Launch`, `Terminate`, `HealthCheck`, or `AZRebalance` process.
- **ECS service scaling:** Application Auto Scaling adjusts the task count on CPU, memory, or `ALBRequestCountPerTarget`. On EC2-backed clusters, capacity providers grow the instances underneath, so there are two layers to check.
- **Lambda:** no policies. Throttling means reserved concurrency or the account limit. Provisioned concurrency keeps environments warm.
- **Caching is scaling:** CloudFront offloads the origin (tune TTLs so cacheable dynamic responses are cached). ElastiCache sits in front of the database.
  - **Redis:** replication, automatic failover, persistence; cluster mode shards writes.
  - **Memcached:** multi-threaded, horizontal, ephemeral, no replication.
  - A falling `CacheHitRate` means check evictions and memory. **DAX** caches DynamoDB reads.
- **RDS:**
  - Changing the instance class restarts the instance; Multi-AZ shortens that by modifying the standby first and failing over.
  - **Storage autoscaling** grows toward a maximum you set and never shrinks.
  - Read replicas are asynchronous. Alarm on `ReplicaLag`, and route reads to replicas explicitly.
- **Aurora:** shared auto-growing storage and up to 15 replicas with minimal lag, load-balanced by the **reader endpoint**. **Serverless v2** scales in ACUs between a floor and ceiling for spiky load.
- **DynamoDB:**
  - On-demand suits spiky, new, or unknown traffic.
  - Provisioned plus auto scaling (target tracking on consumed capacity) is cheaper for steady load.
  - Steady throttling means raise capacity or the ceiling. Bursty throttling means switch to on-demand.
  - Throttling with spare table capacity means a **hot partition**. Fix the key design; use DAX for hot reads.
- **Scalability vs availability:** Multi-AZ adds resilience, not capacity. Decide which one the stem asks for before answering.

📖 Full lesson: [Auto Scaling and Elasticity: EC2, Caching, and Database Scaling](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-auto-scaling-elasticity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 2.2 Implement highly available and resilient environments

|            | ALB                                         | NLB                                           |
| ---------- | ------------------------------------------- | --------------------------------------------- |
| Layer      | 7 (HTTP/HTTPS)                              | 4 (TCP, UDP, TLS)                             |
| Routing    | Path, host, header, query string; redirects | By listener port only                         |
| Addressing | DNS name; IPs change                        | Static IP per AZ, Elastic IP support          |
| Client IP  | `X-Forwarded-For` header                    | Preserved on the connection                   |
| Cross-zone | On by default                               | Off by default; cross-AZ charges when enabled |

- An NLB can front an ALB to give an HTTP stack a fixed IP while keeping layer-7 rules.
- **Target group** = targets + one health check config. One bad setting takes out every target at once.
- **Deregistration delay** (default 300 s) lets in-flight requests finish while the target drains. Deploys that hang "waiting on the load balancer" usually trace to this timer.
- **Health check settings:** protocol, port, path, interval, timeout, and healthy/unhealthy thresholds (consecutive results). Success matcher defaults to 200, so a 301/302/404 on the check path counts as a failure.
- **Unhealthy target:** the LB stops sending new requests and keeps probing. It never terminates anything; replacement needs an ASG with the ELB health check type. If **all** targets are unhealthy, the ALB fails open and routes to all of them.
- **All targets unhealthy after a deploy is systemic:** check the path moved or removed, a port/protocol mismatch or HTTP-to-HTTPS redirect, a security group not allowing the LB's security group on the check port, or an app returning 5xx everywhere. **Some** unhealthy is instance-level.
- **LB error codes:** 502 = bad response or reset from the target. 503 = no registered or no healthy targets available. 504 = target too slow.
- **Route 53 health check types:**
  - **Endpoint:** HTTP, HTTPS, or TCP from global checkers, with optional string matching.
  - **Calculated:** combines child checks, such as "healthy if at least 2 of 3".
  - **Alarm-based:** takes its state from a CloudWatch alarm. This is the only way to cover **private** resources, because the checkers sit outside your VPC.
- **Failover routing:** a PRIMARY record with a health check plus a SECONDARY record (often a static site). For alias records to AWS resources, enable **Evaluate Target Health** instead of paying for a check. "Never fails over" means the health check or that flag is missing.
- **Two layers:** ELB heals inside a Region within seconds, with no DNS change. Route 53 moves users between Regions, but only as fast as TTL caching allows, so keep failover TTLs low. **Application Recovery Controller** adds readiness checks and routing controls for deliberate Regional failover.
- **RDS availability options:**

|             | Multi-AZ instance standby | Read replica     | Multi-AZ DB cluster       |
| ----------- | ------------------------- | ---------------- | ------------------------- |
| Replication | Synchronous               | Asynchronous     | Writer + two standbys     |
| Readable    | No                        | Yes              | Yes, standbys serve reads |
| Failover    | Automatic, same endpoint  | Manual promotion | Automatic                 |

- An ElastiCache Redis replication group with Multi-AZ promotes a replica automatically.
- **Static stability:** survive an AZ loss with capacity already in place.
  - **Recovers automatically:** load balancer rerouting, ASG instance replacement, RDS Multi-AZ promotion, Redis replica promotion.
  - **Needs a human:** promoting a read replica, repointing DNS that has no health check, launching capacity that was never provisioned.
- **Single-AZ trap:** one NAT gateway shared across AZs. Use one per AZ with local routes, and put interface endpoint ENIs in every AZ used.

📖 Full lesson: [High Availability: ELB and Route 53 Health Checks, Multi-AZ Design](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-high-availability-health-checks/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 2.3 Implement backup and restore strategies

- **RPO** = how much data you can lose (age of the newest usable copy). **RTO** = how long you can be down. Convert the stem into both numbers, drop options that miss either one, then pick the cheapest that remains.
- **Multi-AZ is not a backup.** A bad `DELETE` or corruption reaches the standby immediately.
- **AWS Backup** is the default for automating backups across services or accounts.
  - **Plan** = rules with a schedule, retention, and a lifecycle to cold storage.
  - Recovery points live in an encrypted **vault**. **Vault Lock** makes them undeletable before retention ends, the answer for ransomware or malicious-deletion stems.
  - **Tag-based assignment** picks up new resources automatically.
  - Copies can go to another Region (DR) or another account (isolation).
  - It covers EC2, EBS, RDS/Aurora, DynamoDB, EFS, FSx, and S3.
- **Native options:**

| Mechanism                 | Scope                                    | Retention / restore window                                       |
| ------------------------- | ---------------------------------------- | ---------------------------------------------------------------- |
| Data Lifecycle Manager    | EBS snapshots by tag                     | Your schedule. Use it for EBS-only scenarios                     |
| RDS automated backups     | Daily snapshot + transaction logs → PITR | Up to 35 days (0 disables). Removed with the instance by default |
| RDS manual snapshot       | On demand                                | Until you delete it; survives instance deletion                  |
| DynamoDB PITR             | Continuous                               | Any second in the last 35 days                                   |
| DynamoDB on-demand backup | Full snapshot                            | Until deleted                                                    |
| FSx                       | Automatic daily + user-initiated backups | Configurable / until deleted                                     |

- **Every restore makes a new resource.** RDS PITR or snapshot restore gives a new instance with a **new endpoint**. DynamoDB restores to a new table, and EBS to a new volume. "Restored but the app still shows old data" or "cannot connect" means the app was never repointed. A Route 53 CNAME in front of the database makes this a one-record change. DynamoDB restores do not bring auto scaling settings, tags, or alarms.
- **Test restores.** Timing a restore on real data volume is the only proof the RTO holds.
- **S3 versioning:**
  - An overwrite creates a new current version.
  - A delete adds a **delete marker**. Remove the marker to undelete; copy an old version back to undo an overwrite.
  - Versioning can be suspended, never disabled.
  - Add a noncurrent-version lifecycle rule to control cost. MFA Delete guards permanent deletes.
- **Replication (CRR/SRR):** needs versioning on both buckets and copies only new objects. Use **Batch Replication** for existing ones. Replication copies mistakes too, so it gives a second location, not history.
- **FSx for Windows** shadow copies let users restore earlier file versions.
- **DR patterns (cost rises, RTO/RPO fall):**

| Pattern                  | Running in the recovery Region     | RTO             | Cue                                         |
| ------------------------ | ---------------------------------- | --------------- | ------------------------------------------- |
| Backup and restore       | Nothing but backups                | Hours           | "Lowest cost"                               |
| Pilot light              | Live replicated data; app tier off | Tens of minutes | "Data current, servers started at failover" |
| Warm standby             | Full stack, scaled down            | Minutes         | "Scaled-down copy already running"          |
| Multi-site active/active | Full stack serving traffic         | Near zero       | "No downtime"                               |

- **Failover steps:** Route 53 failover routing, then promote a cross-Region read replica (fast, near-current data) or restore a cross-Region copy (slower, older data), repoint apps, scale up, verify.
- **Worked cue:** RPO 15 minutes, RTO 1 hour, lowest cost → pilot light with a cross-Region replica and a rehearsed runbook.

📖 Full lesson: [Backup, Restore, and Disaster Recovery Strategies on AWS](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-backup-restore-disaster-recovery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

[← Back to the study guide](../README.md)
