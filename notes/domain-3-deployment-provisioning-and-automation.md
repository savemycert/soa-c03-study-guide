# Domain 3: Deployment, Provisioning, and Automation (22%)

This domain covers building infrastructure from repeatable definitions (images, CloudFormation and CDK stacks, StackSets) and running day-two operations through Systems Manager and event-driven services instead of SSH and hand-run scripts. Most questions give a failed stack, a drifted resource, or an operational requirement and ask for the mechanism that fits.

## 3.1 Provision and maintain cloud resources

- **AMIs:**
  - Creating an AMI reboots the source instance by default for a consistent filesystem. No-reboot risks an inconsistent image.
  - AMIs are regional, so copy them to each Region you launch in, and share them by account ID or across the organization.
  - **Deregistering does not delete the backing snapshots.** Cleanup is two steps, or storage charges continue.
- **EC2 Image Builder:** a pipeline of source image → build components → test components → distribution (Regions and accounts). Run it on a schedule to keep golden images patched with no manual rebuilds.
- **Container images:** push to ECR for ECS and EKS. ECR lifecycle policies expire old or untagged images.
- **Template sections:**

| Section                           | Operator's use                                                        |
| --------------------------------- | --------------------------------------------------------------------- |
| Resources (only required section) | Logical ID, type, properties                                          |
| Parameters                        | Per-environment values at deploy time                                 |
| Mappings                          | Static lookups, e.g. Region → AMI ID                                  |
| Conditions                        | Create resources only when true (e.g. prod-only Multi-AZ)             |
| Outputs                           | Surface values; `Export` + `Fn::ImportValue` share them across stacks |

- **Cross-stack trap:** you cannot delete a stack, or remove an export, while another stack imports it.
- CloudFormation orders creation by references and `DependsOn`, and runs independent resources in parallel.
- **Change sets** preview Add, Modify, and Remove before execution. Look for **Replacement**: an immutable property change deletes and recreates the resource. That is harmless for a web server and destructive for a database.
- **Protecting stateful resources:**
  - A **stack policy** denies updates to named logical IDs until you override it.
  - **DeletionPolicy** can be `Delete`, `Retain`, or `Snapshot` (EBS, RDS, ElastiCache).
  - **Termination protection** blocks deleting the whole stack.
- **Drift detection:** reports `IN_SYNC`, `MODIFIED`, or `DELETED` per resource and `DRIFTED` for the stack. It never reverts anything, so reconcile the template or the resource before the next update.
- **Stack states:**

| State                      | What happened                                                   | Recovery                                                              |
| -------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------------- |
| `ROLLBACK_COMPLETE`        | Create failed and was rolled back                               | Cannot update. Fix the cause, delete, recreate                        |
| `UPDATE_ROLLBACK_COMPLETE` | Update failed; prior config restored                            | Stack is usable                                                       |
| `UPDATE_ROLLBACK_FAILED`   | Rollback itself failed (out-of-band change, missing dependency) | Fix, then **continue update rollback**, optionally skipping resources |

- **Nested stacks:** a child failure rolls back the parent. Read the child's own events.
- **Troubleshooting a failed deploy:** read the stack events and find the **first** failure. Later ones are cascade noise.
  - **Insufficient free addresses:** the subnet is too small. Every subnet loses 5 addresses, so a /28 has 11 usable. Fix with a bigger CIDR or more subnets, not a retry.
  - **Resource already exists:** an explicit name collides with an existing resource. Let CloudFormation generate names.
  - **Access denied:** the call ran as the deployer, or as the stack's **service role** if one is attached. Check whichever identity actually made it.
- **Multi-account provisioning:**

| Service         | Does                                                                    | Pick when                              |
| --------------- | ----------------------------------------------------------------------- | -------------------------------------- |
| StackSets       | Deploys one template as stacks across accounts and Regions              | Every account needs the same stack     |
| AWS RAM         | Shares one existing resource (subnets, Transit Gateway, Resolver rules) | Many accounts should use one resource  |
| Service Catalog | Portfolio of approved CloudFormation products for self-service          | Users may launch only vetted templates |

- **StackSets permissions:** self-managed means you create the admin and execution roles. **Service-managed** uses Organizations, creates the roles for you, and enables **automatic deployment** to accounts that join a target OU. That is the answer for "every new account gets the baseline".
- **Deployment strategies:**
  - **In-place:** fast and cheap, but rollback means redeploying the old version.
  - **Blue/green:** a parallel environment and a traffic shift at the LB or DNS. Near-instant rollback, double cost during the switch.
  - **ASG instance refresh:** rolls a new launch template version (new AMI) through the group while holding a **minimum healthy percentage**. It also fixes instances left on old template versions.
- **Pipeline chain:** Image Builder → new launch template version → instance refresh.
- **CDK:** code in TypeScript, Python, and others. `cdk synth` produces a CloudFormation template and `cdk deploy` runs a normal stack, so change sets, rollbacks, and drift all behave the same.
- **Terraform vs CloudFormation:**

|         | CloudFormation            | Terraform                                                      |
| ------- | ------------------------- | -------------------------------------------------------------- |
| State   | Held by AWS in the stack  | A state file you store, secure, and lock (often an S3 backend) |
| Scope   | AWS                       | Multi-cloud                                                    |
| Preview | Change set                | Plan                                                           |
| Drift   | On-demand drift detection | State refresh at plan time                                     |

- **Git** is the source of truth. "Versioned, reviewed, auditable infrastructure changes" means a repository workflow, not console edits.

📖 Full lesson: [CloudFormation, StackSets, and golden images: provisioning for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-cloudformation-provisioning-stacksets/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 3.2 Automate the management of existing resources

- **Managed instance prerequisites** (an instance missing from Systems Manager fails one of these):
  1. SSM Agent installed and running. Custom or minimal AMIs may lack it.
  2. An instance profile with `AmazonSSMManagedInstanceCore`.
  3. Outbound HTTPS to the Systems Manager endpoints (`ssm`, `ssmmessages`, `ec2messages`) through an IGW or NAT, or through **interface endpoints** in private subnets. The endpoint security group must allow 443 from the instances.
- **Session Manager:** a shell with no SSH keys, no bastion, and no inbound ports. The agent's outbound channel carries the session. IAM governs access, and session activity can be recorded in CloudWatch Logs or S3. The answer to "close port 22" and "audit interactive access".
- **Run Command:** runs one document (`AWS-RunShellScript`, `AWS-RunPowerShellScript`) across instances targeted by tag. Rate controls cap concurrency and stop on errors, and output can go to S3. One-off only.
- **Automation runbooks:** multi-step workflows calling APIs, scripts, and Run Command, with outputs passed between steps. Predefined ones include `AWS-RestartEC2Instance`, `AWS-StopEC2Instance`, and `AWS-CreateImage`. **Approval steps** add a human gate, and rate controls bound the blast radius. They can be triggered by a maintenance window or an EventBridge rule.
- **Patching campaign with Patch Manager:**

| Piece                  | Defines                                                                                           |
| ---------------------- | ------------------------------------------------------------------------------------------------- |
| Patch baseline         | What counts as patched: classifications, severities, explicit approve/reject, auto-approval delay |
| Patch group (tag)      | Which instances use which baseline                                                                |
| Maintenance window     | When: schedule, duration, cutoff                                                                  |
| `AWS-RunPatchBaseline` | How: **Scan** reports only, **Install** applies (and reboots as configured)                       |
| Compliance data        | Proof of which instances lack approved patches                                                    |

- Typical rhythm: Scan daily, Install inside the window with concurrency and error thresholds.
- **State Manager:** an association of document + targets + schedule that keeps reapplying to fix drift, including on instances launched later. Examples: keep the CloudWatch agent installed, or run `AWS-UpdateSSMAgent`.
- **Inventory** records installed software, versions, and OS details. **Compliance** reports patch and association status.
- **Parameter Store:** hierarchical, versioned config (AMI IDs, URLs, flags). `SecureString` values are KMS-encrypted. Runbooks, documents, and templates resolve parameters at run time, so one update reaches every consumer.
- **Picking the tool:**

| Need                                         | Tool                                         |
| -------------------------------------------- | -------------------------------------------- |
| Run it once across the fleet                 | Run Command                                  |
| Multi-step procedure, maybe approval-gated   | Automation runbook                           |
| Keep a setting true forever                  | State Manager                                |
| OS patches with approval rules and reporting | Patch Manager + maintenance window           |
| Interactive shell                            | Session Manager (never an automation answer) |

- **EventBridge as the router:** a rule matches an event pattern or a schedule and targets Lambda, runbooks, Run Command, SQS, SNS, or Step Functions. Typical pairings:
  - Instance state change → tagging or remediation runbook.
  - Health event → on-call topic.
  - Config compliance change → remediation.
  - GuardDuty finding → isolation runbook.
- **S3 Event Notifications:** object created/removed events, filterable by key prefix or suffix, sent to Lambda, SQS, or SNS. **Enable EventBridge on the bucket** for richer matching, several consumers, or other targets.
- **Lambda** handles custom logic. **Step Functions** handles branching, multi-step flows with retries. **AWS DevOps Agent** is named in the guide as an example for event-driven remediation; know it by name only.
- **Proven chains:**
  - EventBridge schedule → `AWS-StopEC2Instance` at night and `AWS-StartEC2Instance` in the morning on tagged dev instances.
  - New AMI ID written to a parameter → launch templates read it.
- **Selection rule:** managed or declarative beats custom code when the stem says "least operational overhead". Map the verb: once (Run Command), keep enforced (State Manager), multi-step (runbook), when X happens (EventBridge).

📖 Full lesson: [Systems Manager and event-driven automation: managing resources for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-systems-manager-event-driven-automation/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

[← Back to the study guide](../README.md)
