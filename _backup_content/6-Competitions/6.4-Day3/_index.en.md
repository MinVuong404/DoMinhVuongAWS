---
title: "6.4. Day 3: Prove It Arena"
date: 2026-08-23
weight: 4
chapter: false
pre: " <b> 6.4. </b> "
---

# 6.4. Day 3: Prove It Arena — Live Security Simulation & Rapid Remediation

### 1. Grand Finale Arena Problem Statement
The championship **Prove It Arena** plunged competitors into an emergency cascading infrastructure crisis in real time:
* An online financial payment gateway was undergoing a catastrophic Distributed Denial-of-Service (**Layer 7 HTTP DDoS**) assault exceeding 50,000 requests/second dispatched from global botnets.
* Upstream API Gateways throttled (HTTP 429), Lambda compute concurrency quotas were exhausted, and database IOPS experienced severe starvation.
* **Objective:** Within a 90-minute window, competitors were mandated to restore service health to `Healthy`, neutralize adversarial floods, and maintain legitimate customer transaction latencies below 100 ms.

---

### 2. Crisis Remediation Strategy & Technical Execution

```
[ Adversarial Botnet Traffic 50k req/s ]
                  │
                  ▼
  [ 1. Deploy AWS WAF Shield ] ──► (Drops 98% malicious botnet traffic at perimeter)
                  │ (Legitimate traffic 500 req/s)
                  ▼
   [ 2. API Gateway HTTP API ]
                  │
                  ▼
      [ 3. AWS Lambda Compute ] ◄── (Configured Reserved Concurrency = 500)
                  │
                  ▼
  [ 4. Amazon DynamoDB Database ] ◄── (Switched to On-Demand Auto-Scaling Mode)
```

#### Step 1: Deploy AWS WAF with Aggressive Rate-Limiting
Provision a Regional Web ACL enforcing an aggressive limit of 100 requests per 5 minutes per IP:
```bash
aws wafv2 create-web-acl \
    --name EmergencyDDoSMitigationACL \
    --scope REGIONAL \
    --default-action Allow={} \
    --rules '[{
        "Name": "RateLimitRule",
        "Priority": 1,
        "Statement": {
            "RateBasedStatement": {
                "Limit": 100,
                "AggregateKeyType": "IP"
            }
        },
        "Action": { "Block": {} },
        "VisibilityConfig": {
            "SampledRequestsEnabled": true,
            "CloudWatchMetricsEnabled": true,
            "MetricName": "RateLimitRuleMetric"
        }
    }]' \
    --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=WebACLMetric \
    --region ap-southeast-1
```
*Impact:* Within 2 minutes of attaching WAF, 98.5% of malicious junk traffic was dropped at the network edge, relieving downstream Lambda workers.

#### Step 2: Allocate Lambda Reserved Concurrency
Protect account-wide concurrency limits from noisy-neighbor starvation:
```bash
aws lambda put-function-concurrency \
    --function-name PhishingDetectionXGBoostContainer \
    --reserved-concurrent-executions 500 \
    --region ap-southeast-1
```

#### Step 3: Transition DynamoDB to On-Demand Auto-Scaling
Switch NoSQL storage billing mode from Provisioned to On-Demand to accommodate traffic spikes:
```bash
aws dynamodb update-table \
    --table-name PhishingDetectionLogs \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1
```

---

### 3. Deliverables & Technical Takeaway
* **System Recovery Metrics:**
  * Application error rate plummeted from 89.4% down to **0.02%**.
  * Legitimate end-user transaction latency normalized from 4800 ms to **14.2 ms**.
  * Total recovery playbook executed in 52 minutes (against the 90-minute allowance).
* **Architectural Lesson:** Modern cloud resiliency demands **Defense-in-Depth**: Coupling edge filtering (AWS WAF) with dynamic serverless elasticity guarantees operational survival during large-scale adversarial disruptions.
