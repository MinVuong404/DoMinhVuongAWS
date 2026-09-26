---
title: "6.2. Day 1: Govern"
date: 2026-08-21
weight: 2
chapter: false
pre: " <b> 6.2. </b> "
---

# 6.2. Day 1: Govern — Identity Governance & Security Boundaries

### 1. Challenge Scenario Problem Statement
A simulated multinational financial institution account triggered critical governance alerts:
* A public-facing EC2 instance located within a public subnet had been erroneously assigned an IAM Instance Profile granting full `AdministratorAccess`.
* Three administrative IAM Users were operating without MFA protection, holding active access keys unrotated for over 180 days.
* Attack Vector: Malicious actors could exploit Server-Side Request Forgery (SSRF) vulnerabilities to exfiltrate temporary credentials from the instance metadata service (`http://169.254.169.254/latest/meta-data/iam/security-credentials/`), leading to complete account compromise.

---

### 2. Remediation Strategy & Technical Execution

#### Step 1: Audit and Isolate Over-Privileged IAM Roles
Query attached policies using the AWS CLI:
```bash
aws iam list-attached-role-policies --role-name CompromisedWebServerRole
```
*Identified Vulnerability:* Policy `arn:aws:iam::aws:policy/AdministratorAccess` directly bound.

Detach administrator access and scope down strictly to read-only asset fetching:
```bash
# 1. Detach AdministratorAccess
aws iam detach-role-policy \
    --role-name CompromisedWebServerRole \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 2. Attach scoped least-privilege policy
aws iam attach-role-policy \
    --role-name CompromisedWebServerRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

#### Step 2: Enforce IMDSv2 to Mitigate SSRF Exploitation
Mandate session-oriented token handshakes via Instance Metadata Service Version 2 (IMDSv2):
```bash
aws ec2 modify-instance-metadata-options \
    --instance-id i-0123456789abcdef0 \
    --http-tokens required \
    --http-endpoint enabled
```

#### Step 3: Deactivate Compromised / Stale Access Keys
Deactivate stale access keys older than 90 days:
```bash
aws iam update-access-key \
    --user-name finance-analyst-01 \
    --access-key-id AKIAIOSFODNN7EXAMPLE \
    --status Inactive
```

---

### 3. Deliverables & Technical Takeaway
* **Challenge Score:** 100% completion achieved within 35 minutes (well ahead of the 60-minute limit).
* **Architectural Lesson:** Enforcing **IMDSv2** is non-negotiable for cloud computing perimeters, paired with automated IAM Access Analyzer compliance reviews.
