---
title: "Worklog Week 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Worklog Week 5: AWS Lambda Container Deployment & Real-time Inference Latency Optimization

### 1. General Information & Core Objectives
* **Duration:** August 31, 2026 – September 6, 2026 (Week 5).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Provision an **IAM Execution Role** titled `PhishingDetectionLambdaRole` adhering strictly to the *Principle of Least Privilege*.
  2. Instantiate the serverless function **AWS Lambda** `PhishingDetectionXGBoostContainer` from the published Amazon ECR container image.
  3. Execute empirical performance benchmarks across variable memory allocations (128 MB, 256 MB, 512 MB, 1024 MB) to determine the optimal price-performance sweet spot.
  4. Eliminate redundant model deserialization overhead using **Global Scope Pre-loading** techniques (loading model artifacts outside the `lambda_handler` scope).
  5. Configure production execution limits: 5-second timeout, dynamic environment variables (`CONFIDENCE_THRESHOLD`, `DYNAMODB_TABLE`, `SNS_TOPIC_ARN`).

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Created IAM Trust Policy allowing `lambda.amazonaws.com` service principal to assume role.<br>• Attached managed policy `AWSLambdaBasicExecutionRole` for CloudWatch log ingestion. | 08/31/2026 | 08/31/2026 | [AWS Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html) | Successfully created execution role: `arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole`. |
| **Tue** | • Provisioned Lambda function via CLI `aws lambda create-function` with `package-type Image` pointing to ECR URI.<br>• Verified function operational status transitioned to `Active`. | 09/01/2026 | 09/01/2026 | [AWS CLI Lambda create-function](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/lambda/create-function.html) | Function successfully initialized in Singapore (`ap-southeast-1`). |
| **Wed** | • Conducted initial invocation testing measuring cold start latency: Measured 1420 ms due to microVM container image extraction.<br>• Inspected execution metrics via CloudWatch `REPORT` logs. | 09/02/2026 | 09/02/2026 | [Understanding Lambda Cold Starts](https://aws.amazon.com/blogs/compute/operating-lambda-performance-optimization-part-1/) | Identified Python C-extension module imports (`xgboost`, `scipy`) as primary cold start drivers. |
| **Thu** | • Applied architectural optimization: Initialized global variables `MODEL`, `TFIDF`, and `SVD` in global execution scope.<br>• Reused global memory across warm invocations. | 09/03/2026 | 09/03/2026 | [Reusing Execution Context in Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) | Warm inference latency dramatically dropped from 1420 ms to **11 – 14 ms**! |
| **Fri** | • Performed memory benchmarking across tiers:<br>  - 256 MB: Warm latency 35 ms, memory consumed 148 MB.<br>  - 512 MB: Warm latency 12 ms, memory consumed 152 MB (Optimal).<br>  - 1024 MB: Warm latency 11 ms (Doubled vCPU cost with marginal latency gain). | 09/04/2026 | 09/04/2026 | [AWS Lambda Memory and Computing Power](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-common.html) | Finalized **512 MB RAM** and **5-second timeout** as production operational baseline. |
| **Sat - Sun**| • Ran load simulation script dispatching 50 consecutive requests to measure P95 and P99 latencies.<br>• Injected dynamic parameters into Lambda environment variables. | 09/05/2026 | 09/06/2026 | [Configuring Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html) | System achieved exceptional stability with P95 latency holding under 13.2 ms. |

---

### 3. Practical CLI Configurations & Benchmark Telemetry

#### 3.1. IAM Role Setup and Lambda Container Provisioning:
```bash
# 1. Create IAM execution role with trust policy
aws iam create-role \
    --role-name PhishingDetectionLambdaRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": { "Service": "lambda.amazonaws.com" },
                "Action": "sts:AssumeRole"
            }
        ]
    }'

# 2. Attach basic execution policy for CloudWatch logging
aws iam attach-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# 3. Provision Lambda function from ECR container image
aws lambda create-function \
    --function-name PhishingDetectionXGBoostContainer \
    --package-type Image \
    --code ImageUri=803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest \
    --role arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole \
    --memory-size 512 \
    --timeout 5 \
    --environment "Variables={CONFIDENCE_THRESHOLD=0.9,DYNAMODB_TABLE=PhishingDetectionLogs,SNS_TOPIC_ARN=arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic}" \
    --region ap-southeast-1
```

#### 3.2. Real CloudWatch Execution Telemetry:
```text
REPORT RequestId: 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d
Duration: 12.34 ms
Billed Duration: 13 ms
Memory Size: 512 MB
Max Memory Used: 154 MB
Init Duration: 1380.25 ms
```
*(Note: `Init Duration` appears exclusively during initial container cold start provisioning).*

---

### 4. Technical Challenges & Troubleshooting

* **Issue: `Task timed out after 3.00 seconds` during cold start invocation.**
  * *Symptom:* Default Lambda timeout of 3.0 seconds resulted in an HTTP 504 Gateway Timeout during the initial un-cached invocation.
  * *Root Cause Analysis:* During cold start initialization, Lambda must instantiate the Firecracker microVM, fetch the container layers from ECR, launch Python 3.12, and deserialize ~38 MB of models into memory. Cumulative startup duration was 1.4 – 1.8 seconds, which pushed total round-trip time past 3.0 seconds.
  * *Resolution:* Increased timeout threshold from 3 to **5 seconds** via AWS CLI:
    ```bash
    aws lambda update-function-configuration \
        --function-name PhishingDetectionXGBoostContainer \
        --timeout 5 \
        --region ap-southeast-1
    ```
    This accommodates cold starts with headroom, while warm invocations consistently complete in ~12 ms.

---

### 5. Deliverables & Milestones
1. **Fully Operational Lambda Container Function:** Deployed and verified in AWS Singapore region.
2. **Benchmark-Validated Performance:** Sub-15ms warm inference latency consuming only 154 MB RAM within a 512 MB envelope.
3. **Decoupled Architecture:** Application runtime parameters dynamically managed via Lambda Environment Variables.
