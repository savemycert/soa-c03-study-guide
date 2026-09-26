# Domain 5: Networking and Content Delivery (18%)

This domain asks you to build VPC networking correctly, connect networks privately, operate Route 53 and CloudFront, and then diagnose broken paths from logs and metrics. Many questions hinge on a single trigger phrase, such as "no additional cost", "transitive", "static IP", or "ACCEPT then REJECT".

## 5.1 Implement and optimize networking features and connectivity

- **CIDR planning:** VPC /16 to /28; subnets sit in one AZ, never overlap, and cannot be resized. Peered ranges must not overlap. IPAM tracks ranges across accounts.
- **Five reserved IPs per subnet:** network (.0), VPC router (.1), Amazon DNS (.2), future use (.3), broadcast (last). A /28 gives 11 usable addresses.
- **Routing:** each subnet uses exactly one route table (main or custom), the `local` route is permanent, and the **most specific prefix wins**.
- **Public subnet** = a `0.0.0.0/0` route to an IGW **and** public IPs on the instances. There is no public setting on the subnet itself.
- **NAT gateway:** managed and zonal. It lives in a public subnet with an Elastic IP and allows outbound-initiated traffic only. Use one per AZ, with each private subnet routing to its local NAT. A NAT instance is the self-managed distractor.
- **IPv6:** addresses are public and there is no NAT. Outbound-only traffic uses an **egress-only internet gateway** (`::/0`).

| | Security group | Network ACL |
|---|---|---|
| Scope | Network interface | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only, evaluated together | Allow and deny, lowest number first |
| Default | Deny in, allow out | Default NACL allows all; custom NACL denies all |

- NACLs need return rules for **ephemeral ports** (commonly 1024-65535). Use security groups for workload rules and NACLs for subnet-wide blocks such as a hostile CIDR.
- **Endpoints:**
  - A **gateway endpoint** serves only S3 and DynamoDB. It is a route-table entry (managed prefix list), costs nothing, and works only inside its own VPC.
  - An **interface endpoint** (PrivateLink) is ENIs in your subnets for most other services, billed hourly per AZ plus per GB. Private DNS makes the normal hostname resolve privately.
  - Both accept **endpoint policies**.
- **Providing a service over PrivateLink:** NLB + endpoint service; one-way, and overlapping CIDRs do not matter.
- **Peering vs Transit Gateway:**
  - Peering is one-to-one, needs non-overlapping CIDRs and routes on both sides, and is **never transitive**. It suits a few VPCs.
  - Transit Gateway is a hub with transitive routing by default and one attachment per VPC or VPN. Its own route tables (association and propagation) segment traffic, for example spokes reaching shared services but not each other.
- **Site-to-Site VPN:** customer gateway on premises; virtual private gateway (one VPC) or Transit Gateway attachment (many VPCs) on AWS. Each connection has **two tunnels**, so configure both. Routes are static or BGP, with optional route propagation.
- **Client VPN:** managed OpenVPN-based access for individual users (client CIDR pool, subnet associations, authorization rules, certificate or IdP auth). "Office network" means Site-to-Site; "laptops" means Client VPN.
- **Auditing protection in one account:**

| Service | Layer / scope | Audit check |
|---|---|---|
| AWS WAF | L7 web ACLs on CloudFront, ALB, API Gateway (not instances or NLBs) | Is each web ACL actually associated? |
| Shield Standard / Advanced | L3/L4 DDoS; Standard automatic, Advanced paid with response team and cost protection | Which resources are subscribed to Advanced? |
| Network Firewall | Stateful VPC firewall in firewall subnets | Policy **and** route tables steering traffic through it |
| Resolver DNS Firewall | Outbound DNS queries vs domain lists | Rule groups associated per VPC |

- **Cost levers:**
  - Route S3 and DynamoDB through **gateway endpoints** to stop paying NAT data processing; the prefix route beats `0.0.0.0/0`, so traffic shifts at once.
  - Cross-AZ private traffic is charged, same-AZ is not. One shared NAT saves hourly fees but adds cross-AZ charges and a failure domain.
  - Peering has no hourly charge; Transit Gateway bills per attachment and per GB.

📖 Full lesson: [VPC Networking and Private Connectivity for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-vpc-networking-private-connectivity/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 5.2 Configure domains, DNS services, and content delivery

- **Hosted zones:** private zones answer only associated VPCs (DNS resolution and hostnames enabled). Public plus private for one name is split-horizon; the private zone wins inside the VPC.
- **Records:** A/AAAA for addresses, CNAME name-to-name (never at the apex), TXT for verification and SPF/DKIM, MX (lowest priority first). Hosting a zone is separate from registering the domain.
- **Alias records** target CloudFront, ALB/NLB, API Gateway, Global Accelerator, or another record in the zone. They work at the apex, queries to AWS targets are free, Route 53 tracks the target's IP changes, and they can evaluate target health.
- **TTL play for a migration:** lower the TTL (e.g. 3600 → 60) at least one old-TTL period before the cutover, switch, confirm health, then raise it again.
- **Resolver:** at VPC base + 2 (or `169.254.169.253`), reachable only from inside the VPC.

| Direction | Build |
|---|---|
| On premises must resolve VPC / private-zone names | **Inbound** endpoint; on-prem DNS conditionally forwards to its IPs |
| VPC instances must resolve data-center names | **Outbound** endpoint + forwarding rules associated with the VPCs |

- Resolver endpoints carry DNS only. The VPN or other private link must already exist.
- **Routing policies:**

| Policy | Cue |
|---|---|
| Simple | One resource, no health checks |
| Weighted | Canary, gradual migration, A/B |
| Latency | Multi-Region, fastest Region per user |
| Failover | Active-passive with a health check |
| Geolocation | By user country or continent; always add a default record |
| Geoproximity | By distance to resources, with a bias to shift share |
| Multivalue answer | Up to eight healthy records, client picks. Not a load balancer |

- **Canary cutover:** lower the TTL, weight 90/10, watch errors, step to 0/100; roll back by setting the new weight to 0.
- **Query logs:**
  - **Public DNS query logging** records who queries your public zone. It goes to CloudWatch Logs, and the log group must be in us-east-1.
  - **Resolver query logging** records what your instances look up, per VPC, to CloudWatch Logs, S3, or Firehose. It includes DNS Firewall actions.
- **CloudFront origins:** a private S3 bucket behind **Origin Access Control** (the successor to OAI), or custom HTTP origins (ALB, API Gateway, web servers).
- **Cache behaviors:** path patterns evaluated in order, first match wins, so put the most specific first.
  - The **cache policy** sets the cache key and TTL bounds.
  - The **origin request policy** forwards values to the origin without adding them to the key.
- **TTL logic:** origin `Cache-Control`/`Expires` is clamped to the min and max TTL; with no header the default TTL applies.
- **Invalidations:** paths over the monthly free quota are billed, and one wildcard path is billed as a single path. Prefer **versioned file names** and keep invalidations for mistakes.
- **Certificate rule:** a CloudFront custom-domain certificate must be in ACM in **us-east-1**. An ALB certificate lives in the ALB's Region.
- **CloudFront vs Global Accelerator:** caching HTTP content means CloudFront. Two static anycast IPs, TCP/UDP on any port, or failover between Regions with no DNS wait means Global Accelerator, which caches nothing.

📖 Full lesson: [Route 53 DNS and CloudFront Content Delivery for SOA-C03](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-route53-dns-cloudfront-delivery/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

## 5.3 Troubleshoot network connectivity issues

- **Fixed checking order:** route tables (both directions) → security groups → NACLs → gateway/endpoint/attachment state → DNS. Only then suspect the OS firewall or the app, which AWS tools cannot see.
- **Trace every path twice:** forward on the destination port, back on the ephemeral port.
- **No internet access:**
  - Public subnet: check for a missing IGW route or a missing public/Elastic IP.
  - Private subnet: check for a missing `0.0.0.0/0` → NAT route, or a **NAT gateway placed in a private subnet**.
  - NAT metrics: `ErrorPortAllocation` means port exhaustion (add NAT gateways or split traffic). `PacketsDropCount` counts drops.
- **Instances cannot talk:** confirm the source's **actual** security group matches the one the rule references.
- **Flow log basics:** capture at the VPC, subnet, or ENI level, delivered to CloudWatch Logs or S3. Read the addresses, ports, and `action`.

| Signature | Meaning | Fix |
|---|---|---|
| ACCEPT request, REJECT reply, same ENI | Stateless NACL blocking the return (a SG cannot do this) | Outbound ephemeral-port rule on the NACL |
| REJECT on the request | SG or NACL blocking inbound | SG inbound first, then NACL |
| No record at all | Packets never reached the ENI | Routing, DNS, source-side controls |

- **What flow logs leave out:** Amazon DNS queries, metadata (`169.254.169.254`), and DHCP are never logged. In `log-status`, `NODATA` means no traffic and `SKIPDATA` means records were dropped.
- **Peering:** check for missing routes on either side (an asymmetric black hole), an expectation of transitivity, or overlapping CIDRs (peering cannot be created).
- **Transit Gateway:** both the VPC route table and the TGW route table (and their reverse routes) must be right. `PacketsDropCountNoRoute` and `PacketsDropCountBlackhole` locate the drop at the TGW.
- **Reachability Analyzer** checks configuration statically and names the blocking SG, NACL, or route without sending traffic. Use flow logs to see what live traffic did.
- **Which log to read:**

| Question | Log |
|---|---|
| Blocked before the instance? | VPC Flow Logs |
| LB or target at fault? | ELB access logs: compare `elb_status_code` with `target_status_code`; check `target_processing_time` |
| Which WAF rule blocked a user? | WAF web ACL logs: terminating rule ID |
| Edge cache or origin? | CloudFront standard logs: the `x-edge-result-type` and `sc-status` fields |
| What did the app do? | Container logs in CloudWatch Logs |

- **ELB logs:** 504 with `target_processing_time` -1 means the target never answered; ELB 5xx with target 200 means the LB tier is at fault.
- **Low CloudFront hit ratio:** check `x-cache` and `x-edge-result-type`. Mostly `Miss` means a caching problem; `Error` means the origin failed (use origin failover, not cache tuning). Causes:
  - A cache key that includes all query strings, headers, or cookies (tracking parameters).
  - An effectively zero TTL.
  - Invalidating on every deploy.
  - Fix by keying only on what varies the response, aligning TTLs, and versioning object names.
- **VPN:** `TunnelState` 0 means IKE is failing (pre-shared key, customer gateway IP or config, UDP 500/4500 blocked). Tunnel up with no traffic means static routes or BGP propagation, then security groups allowing the on-premises CIDR.
- **Endpoint failures:**
  - Resolves to a public IP: private DNS is off, or VPC DNS support is disabled.
  - Timeouts: the endpoint security group is missing inbound 443.
  - Explicit denies: check the endpoint policy.
  - Gateway endpoint: the subnet's route table is not associated with it.
  - Cross-account PrivateLink: stuck in pending acceptance.
- **Monitoring:** **Network Monitor** probes hybrid paths for loss and latency; **Internet Monitor** covers end users' internet paths.

📖 Full lesson: [AWS Network Troubleshooting: VPC Flow Logs and the Layered Diagnosis Method](https://www.savemycert.com/revision/aws-cloudops-engineer-associate/aws-network-troubleshooting-vpc-flow-logs/?utm_source=github&utm_medium=readme&utm_campaign=soa-c03-study-guide)

[← Back to the study guide](../README.md)
