---
title: "6.3. Day 2: Investigate"
date: 2026-08-22
weight: 3
chapter: false
pre: " <b> 6.3. </b> "
---

# 6.3. Day 2: Investigate — Digital Cloud Forensics & Log Triaging

### 1. Challenge Scenario Problem Statement
Cloud telemetry monitoring alerted on an anomalous spike in outbound internet bandwidth (Data Transfer Out) within the preceding 2 hours:
* Massive traffic volumes originating from within the internal VPC were communicating with an unknown external IP address (`198.51.100.45`).
* Investigation Mandates:
  1. Pinpoint the compromised internal EC2 compute node generating malicious network streams.
  2. Reconstruct the initial compromise event using **AWS CloudTrail** audit logs.
  3. Enforce immediate network quarantine to arrest data exfiltration without destroying in-memory digital evidence.

---

### 2. Forensic Plan & Technical Execution

#### Step 1: Querying VPC Flow Logs with Amazon Athena
Execute SQL queries over ingested network flow records to isolate adversary communications:
```sql
SELECT srcaddr, dstaddr, srcport, dstport, protocol, bytes, action
FROM "vpc_flow_logs_db"."flow_logs"
WHERE dstaddr = '198.51.100.45'
ORDER BY bytes DESC
LIMIT 10;
```
*Investigation Finding:* Internal IP `10.0.2.145` (matching EC2 instance `i-0987654321fedcba0`) had exfiltrated 4.2 GB of data over port 4444 (TCP Reverse Shell).

#### Step 2: Adversary Attribution via AWS CloudTrail
Investigate control plane API calls to establish initial access:
```bash
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=i-0987654321fedcba0 \
    --query 'Events[*].[EventTime, EventName, Username, SourceIPAddress]' \
    --output table
```
*Finding:* Discovered an unauthorized `AuthorizeSecurityGroupIngress` call opening SSH port 22 to `0.0.0.0/0` from an unverified IP 3 hours prior.

#### Step 3: Isolating Compute Node via Quarantine Security Group
Rather than stopping the instance (which flushes volatile RAM evidence), assign an empty quarantine security group with zero inbound/outbound rules:
```bash
# 1. Provision empty quarantine security group
QUARANTINE_SG=$(aws ec2 create-security-group \
    --group-name ForensicQuarantineSG \
    --description "Isolate compromised instance for digital forensics" \
    --vpc-id vpc-0a1b2c3d4e5f6g7h8 \
    --output text --query 'GroupId')

# 2. Revoke default egress rule
aws ec2 revoke-security-group-egress \
    --group-id $QUARANTINE_SG \
    --protocol -1 \
    --cidr 0.0.0.0/0

# 3. Attach quarantine group to compromised instance
aws ec2 modify-instance-attribute \
    --instance-id i-0987654321fedcba0 \
    --groups $QUARANTINE_SG
```

---

### 3. Deliverables & Technical Takeaway
* **Challenge Score:** 100% forensic identification and quarantine accomplished in 40 minutes.
* **Architectural Lesson:** Adhere to standardized incident response procedures: Preserving volatile memory states for disk snapshotting is crucial prior to terminating compromised instances.
