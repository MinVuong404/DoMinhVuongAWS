---
title: "DynamoDB & Amazon SNS"
date: 2026-08-25
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# 4.5. Amazon DynamoDB Audit Persistence & Amazon SNS Alerting

### 4.5.1. Provisioning Amazon DynamoDB NoSQL Table

**Amazon DynamoDB** serves as the immutable audit repository recording telemetry for every scanned email transaction, supporting compliance verification and future retraining data pipelines.

Execute the following AWS CLI command to provision table `PhishingDetectionLogs` under Provisioned Capacity mode (1 RCU / 1 WCU) to maximize **AWS Free Tier lifetime coverage**:
```bash
aws dynamodb create-table \
    --table-name PhishingDetectionLogs \
    --attribute-definitions AttributeName=request_id,AttributeType=S \
    --key-schema AttributeName=request_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region ap-southeast-1
```

Confirm table status until `ACTIVE`:
```bash
aws dynamodb describe-table \
    --table-name PhishingDetectionLogs \
    --query 'Table.[TableStatus, TableArn]' \
    --output text \
    --region ap-southeast-1
```
*Capture Table ARN:* `arn:aws:dynamodb:ap-southeast-1:803146828520:table/PhishingDetectionLogs`.

---

### 4.5.2. Amazon SNS Topic Setup & Email Incident Subscription

**Amazon SNS (Simple Notification Service)** provides an asynchronous Pub/Sub messaging bus, decoupling emergency alert dispatches to Security Operations Center (SOC) personnel without adding round-trip latency to client executions.

#### Step 1: Provision SNS Topic
```bash
aws sns create-topic \
    --name PhishingAlertTopic \
    --region ap-southeast-1
```
*Capture Topic ARN:* `arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic`.

#### Step 2: Subscribe Administrative Email Address
```bash
aws sns subscribe \
    --topic-arn arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region ap-southeast-1
```

{{% notice warning %}}
Access inbox `vuong.dmforwork@gmail.com` and click **Confirm subscription** inside the AWS verification email to activate the alert delivery channel.
{{% /notice %}}

---

### 4.5.3. Scoped Least Privilege IAM Permission Attachment

Create granular permission policy file `lambda-dynamodb-sns-policy.json`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowDynamoDBLogging",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:GetItem"
            ],
            "Resource": "arn:aws:dynamodb:ap-southeast-1:803146828520:table/PhishingDetectionLogs"
        },
        {
            "Sid": "AllowSNSPublishAlert",
            "Effect": "Allow",
            "Action": "sns:Publish",
            "Resource": "arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic"
        }
    ]
}
```

Attach the inline policy to `PhishingDetectionLambdaRole`:
```bash
aws iam put-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-name DynamoDBAndSNSPolicy \
    --policy-document file://lambda-dynamodb-sns-policy.json
```

---

### 4.5.4. High-Risk Alert Simulation & DynamoDB Verification

Transmit an urgent banking phishing lure payload via API Gateway:
```bash
curl -X POST https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
    -H "Content-Type: application/json" \
    -d '{
        "email_text": "URGENT SECURITY ALERT: Unauthorized login detected from Russia on your Techcombank corporate account. Click https://techcombank-security-otp.cc to cancel fraudulent $2,500 wire transfer immediately!"
    }'
```

*API Gateway JSON response:*
```json
{
    "request_id": "a4f89d12-612b-4e01-9a73-8cb23e41b0fa",
    "prediction": "Phishing",
    "confidence": 0.9912,
    "confidence_percent": "99.12%",
    "risk_level": "CRITICAL",
    "inference_latency_ms": 13.12
}
```

#### 1. Verify DynamoDB Audit Entry:
```bash
aws dynamodb get-item \
    --table-name PhishingDetectionLogs \
    --key '{"request_id": {"S": "a4f89d12-612b-4e01-9a73-8cb23e41b0fa"}}' \
    --region ap-southeast-1 \
    --output json
```

*Verified item attributes:*
```json
{
    "Item": {
        "request_id": { "S": "a4f89d12-612b-4e01-9a73-8cb23e41b0fa" },
        "prediction": { "S": "Phishing" },
        "confidence": { "S": "0.9912" },
        "risk_level": { "S": "CRITICAL" },
        "latency_ms": { "S": "13.12" },
        "timestamp": { "S": "2026-08-25T14:30:15.120Z" },
        "text_preview": { "S": "URGENT SECURITY ALERT: Unauthorized login detected from Russia..." }
    }
}
```

#### 2. Confirm Amazon SNS Email Delivery:
Within 1.5 seconds, administrator inbox `vuong.dmforwork@gmail.com` receives the incident dispatch:
* **Subject:** `[URGENT SECURITY] High-Confidence Phishing Email Detected (99.1%)`
* **Body:** Contains Request ID trace, 99.12% confidence, CRITICAL risk rating, and email snippet for rapid security triage.
