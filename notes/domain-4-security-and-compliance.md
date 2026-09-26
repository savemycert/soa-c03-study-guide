# Domain 4: Security and Compliance (16%)

This domain looks at security from the operator's seat: getting IAM and multi-account guardrails right, proving who did what, and keeping data encrypted and findings remediated. Questions are mostly triage (why is this denied?) and service choice (which tool finds this problem?).

## 4.1 Implement and manage security and compliance tools and policies

- **Password policy:** length, character rules, reuse, expiry. It covers IAM user console passwords only, not access keys, federated users, or root. It **cannot require MFA**.
- **Enforcing MFA:** a `Deny` on everything except MFA self-management when `aws:MultiFactorAuthPresent` is not true, using `BoolIfExists` so access-key calls (no MFA context) are denied too. Secure the root user's MFA first.
- **Roles, not keys:** instance profiles for EC2, task and execution roles for ECS and Lambda, and `sts:AssumeRole` for other accounts. Any answer that bakes keys into an AMI, user data, or shared config is wrong when a role exists.
- **Federation:** IAM Identity Center (permission sets become roles in each account, one portal) is the low-effort multi-account choice. Per-account SAML 2.0 trust works but must be maintained account by account.
- **Policy families:** identity-based policies attach to a principal. Resource-based policies (bucket, queue, topic, key) name a `Principal`.
  - Same account: an allow from either one is enough.
  - **Cross-account: both sides must allow.** A resource policy lets the caller keep its own identity.
- **Conditions to know:**
  - `aws:SourceIp` limits calls to a corporate CIDR. It checks public IPs, so it will not match calls arriving through a VPC endpoint.
  - `aws:PrincipalOrgID` goes in resource policies: allow the whole organization without listing accounts.
  - `aws:RequestedRegion` controls which Regions calls may target.
- **Permissions boundary:** caps one user or role and grants nothing. The effective permissions are the intersection with the identity policy. Use it so teams can create roles without escalating.
- **Evaluation:**
  1. An explicit deny anywhere ends evaluation.
  2. The SCP, boundary, and session policy must each allow the action.
  3. An identity or resource policy must grant it.
- **AccessDenied triage (in order):**
  1. The actual principal (instance role? container falling back to the instance role instead of its task role? check `userIdentity` in CloudTrail).
  2. The exact action and ARN. `s3:PutObject` needs `bucket/*`, not just the bucket ARN.
  3. The identity policy.
  4. Explicit denies in the resource policy (TLS-only, VPC endpoint-only conditions).
  5. The SCP or boundary.
  6. Confirm with the **policy simulator** before and after the fix.
- **Audit toolkit:**

| Question | Tool |
|---|---|
| Who did it, from where, when | CloudTrail (`userIdentity`, `eventName`, source IP) |
| Would this policy allow it | IAM policy simulator (nothing executes) |
| What is reachable from outside the account or org | IAM Access Analyzer external access findings |
| What access is never used | Access Analyzer unused-access findings, access advisor |
| Password and key age, MFA status per user | Credential report |

- **Organizations and SCPs:**
  - OUs are organized by function (workloads, sandbox, security).
  - SCPs **never grant**. "The SCP allows S3 but the user cannot" just means no IAM policy grants it.
  - An action must be allowed at the root, the OU, and the account.
  - SCPs **do not apply to the management account**, so keep workloads out of it.
- **Control Tower** packages Organizations, Identity Center, and guardrails into a landing zone.
- **Trusted Advisor security checks:**

| Finding | Fix |
|---|---|
| Security group open to 0.0.0.0/0 on risky ports | Remove or narrow the rule |
| Root account without MFA | Add an MFA device to root |
| Open S3 bucket permissions | Block Public Access + correct the bucket policy |
| Old or unused access keys | Rotate or delete; move to roles |
| Working as root (IAM use check) | Use IAM principals |

- Core checks are available to everyone; Business and Enterprise support unlock the rest. Status changes reach EventBridge, which can trigger SNS, Lambda, or a runbook.
- **Region and service restriction:** an SCP denying all actions when `aws:RequestedRegion` is outside the approved list, with `NotAction` exempting global services (IAM, CloudFront, Route 53). Only an SCP reaches every principal in every member account. Deny unapproved services the same way.
- **AWS Config:** the recorder tracks configuration history, and rules mark resources compliant or noncompliant. Try managed rules before Lambda-backed custom rules. **Auto-remediation** attaches an SSM Automation runbook. **Conformance packs** bundle rules and remediations for deployment across accounts and Regions. Results feed Security Hub standards.

📖 Full lesson: [IAM, AWS Organizations, and Compliance Tools for CloudOps (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-iam-organizations-compliance-tools/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 4.2 Implement strategies to protect data and infrastructure

- **Classification drives controls:** for example public / internal / confidential. Confidential data gets customer managed keys, tight resource policies, logging, and monitoring. Carry the tier in a tag such as `DataClassification` so Config rules and policies can target it.
- **Macie** finds PII and credentials inside S3 objects, through automated discovery sampling or targeted jobs. It also flags public or unencrypted buckets. Findings go to EventBridge and Security Hub.
- **KMS operations:**
  - **Default EBS encryption** is set per account per Region and covers new volumes and snapshot copies only.
  - The **key policy** is the primary authority. An IAM allow does nothing unless the key policy permits the principal or delegates to IAM.
  - Cross-account key use needs the key policy **and** the caller's IAM policy.
  - **Grants** give programmatic, often service-created access.
  - **Automatic rotation** on customer managed keys keeps old material so existing ciphertext still decrypts.
- **KMS AccessDenied order:** key policy → IAM (`kms:Decrypt`, `kms:GenerateDataKey`) → grants. An instance that cannot use an encrypted volume is usually missing KMS permissions, not EBS permissions.
- **Encrypting what already exists (no in-place switch):**
  1. Snapshot the volume or DB instance.
  2. Copy the snapshot with encryption on and the chosen key. This copy is the pivot step.
  3. Restore a new volume or instance from the copy.
  4. Cut over in a window; writes after the snapshot need re-syncing.
  5. Verify, then delete the old resource.
- **S3:** default encryption is SSE-S3. Pick SSE-KMS for your own key, key-level access control, and CloudTrail records of key use. Changing the default affects new writes only.
- **ACM:**
  - Free public certificates for ELB, CloudFront, and API Gateway; you never touch the private key.
  - With **DNS validation** they renew automatically while the CNAME stays in place. Email validation needs a person at renewal.
  - **Expired certificate causes:** the validation record was deleted, or the certificate was **imported** (imported certificates never auto-renew, so monitor expiry).
- **Enforcing TLS:** an S3 bucket policy denying requests where `aws:SecureTransport` is `false`. On the ALB, make the port 80 listener redirect to HTTPS. "Encrypted at every hop" means re-encrypting to HTTPS targets rather than ending TLS at the LB.
- **Secrets:** never in code, user data, AMIs, or plain env files. Fetch them at run time with the workload's role.

| | Secrets Manager | Parameter Store SecureString |
|---|---|---|
| Rotation | Automatic; managed integration for RDS | None built in |
| Cost | Per secret per month | Standard tier free |
| Pick when | Credentials must rotate | Static secrets or config, cost-sensitive |

- **Findings services:**

| Question | Service |
|---|---|
| Is something attacking me now? | GuardDuty (CloudTrail, VPC Flow Logs, DNS logs; no agents; detects, never blocks) |
| Known CVEs or unintended reachability on EC2, ECR images, Lambda? | Inspector |
| Sensitive data in S3? | Macie |
| Resources configured per policy? | AWS Config |
| One view, standards score, all accounts and Regions | Security Hub (delegated administrator) |

- **Distractor logic:** Inspector reports what could be exploited, GuardDuty what is being exploited. Security Hub is never the primary detector for a specific symptom. **AWS Security Agent** is named in the guide next to these services; recognize the name.
- **Remediation loop:** a finding goes to EventBridge, where a rule filters by source, type, and severity. It notifies SNS and invokes Lambda or an SSM runbook (for example, swap in an isolation security group and snapshot the volumes, revoke an open rule, or enable Block Public Access).
  - Config rules can call a runbook directly.
  - Security Hub **custom actions** send chosen findings to EventBridge on demand.
  - Automate reversible containment freely, gate destructive actions behind approval, and record what the automation changed.

📖 Full lesson: [Data Protection on AWS: KMS, ACM, Secrets, and Security Services (SOA-C03)](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-data-protection-security-services/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

[← Back to the study guide](../README.md)
