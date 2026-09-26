---
title: "Worklog Week 6"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Worklog Week 6: Amazon DynamoDB NoSQL Audit Logging & Amazon SNS Alert Notification System

### 1. General Information & Core Objectives
* **Duration:** September 7, 2026 – September 13, 2026 (Week 6).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Design an optimized NoSQL schema for **Amazon DynamoDB** table `PhishingDetectionLogs` to record real-time immutable audit trails.
  2. Configure **Provisioned Capacity** (1 RCU / 1 WCU) to capitalize on the **AWS Free Tier lifetime policy** (25 GB free storage and 200 million requests/month).
  3. Provision **Amazon SNS Topic** `PhishingAlertTopic` and configure an automated email subscriber pipeline for incident response.
  4. Author a strictly scoped Least Privilege inline policy for `PhishingDetectionLambdaRole` granting granular `dynamodb:PutItem` and `sns:Publish` actions.
  5. Implement business alert logic: Trigger real-time emergency notifications whenever high-confidence phishing (**≥ 90%**) is identified.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Designed NoSQL key schema: Partition Key `request_id` (String UUIDv4).<br>• Defined entity attributes: `timestamp`, `text_preview`, `prediction`, `confidence`, `risk_level`, `latency_ms`. | 09/07/2026 | 09/07/2026 | [Amazon DynamoDB Core Components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) | Schema ratified by mentor as optimal for high-throughput single-key writes. |
| **Tue** | • Provisioned table `PhishingDetectionLogs` via AWS CLI.<br>• Set Provisioned mode (1 RCU / 1 WCU) and verified KMS Server-Side Encryption. | 09/08/2026 | 09/08/2026 | [AWS CLI DynamoDB create-table](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/dynamodb/create-table.html) | Table status reached `ACTIVE` within 12 seconds with $0.00 ongoing cost. |
| **Wed** | • Created Amazon SNS Topic `PhishingAlertTopic` in `ap-southeast-1`.<br>• Subscribed administrative email `vuong.dmforwork@gmail.com`.<br>• Verified subscription handshake via confirmation link in inbox. | 09/09/2026 | 09/09/2026 | [Amazon SNS Getting Started](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html) | Subscription status transitioned to `Confirmed` with valid ARN. |
| **Thu** | • Drafted Least Privilege IAM Policy: Restricted actions strictly to `dynamodb:PutItem` on `PhishingDetectionLogs` and `sns:Publish` on `PhishingAlertTopic`.<br>• Attached policy to Lambda execution role. | 09/10/2026 | 09/10/2026 | [IAM Policies for Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-identity-based.html) | Eliminated excessive privileges, achieving strict compliance with cloud security baselines. |
| **Fri** | • Updated `lambda_function.py`: Integrated `boto3` SDK for DynamoDB persistence and SNS publishing.<br>• Formatted structured alert emails containing request trace ID, confidence percentage, and threat snippet. | 09/11/2026 | 09/11/2026 | [Boto3 DynamoDB Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html)<br>[Boto3 SNS Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html) | Clean integration completed without degrading core inference latency. |
| **Sat - Sun**| • Executed simulated phishing test: Dispatched banking credential harvesting sample (98.4% confidence).<br>• Verified DynamoDB entry written in 8 ms.<br>• Received alert email in inbox within 1.2 seconds! | 09/12/2026 | 09/13/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) | Confirmed end-to-end integration between NoSQL audit logging and real-time alerting. |

---

### 3. Practical Configurations & Code Artifacts

#### 3.1. DynamoDB & SNS CLI Setup:
```bash
# 1. Provision DynamoDB table under Free Tier limits
aws dynamodb create-table \
    --table-name PhishingDetectionLogs \
    --attribute-definitions AttributeName=request_id,AttributeType=S \
    --key-schema AttributeName=request_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region ap-southeast-1

# 2. Provision SNS Incident Alert Topic
aws sns create-topic \
    --name PhishingAlertTopic \
    --region ap-southeast-1

# 3. Subscribe administrative email endpoint
aws sns subscribe \
    --topic-arn arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region ap-southeast-1
```

#### 3.2. Granular Least Privilege IAM Policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowDynamoDBLogging",
            "Effect": "Allow",
            "Action": "dynamodb:PutItem",
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

#### 3.3. Python Audit Logging & Incident Dispatch Logic:
```python
import boto3
import json
import uuid
import datetime

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')
table = dynamodb.Table('PhishingDetectionLogs')

# 1. Ingest audit item into DynamoDB
table.put_item(
    Item={
        'request_id': request_id,
        'timestamp': datetime.datetime.utcnow().isoformat(),
        'prediction': 'Phishing',
        'confidence': str(confidence),
        'text_preview': email_text[:250],
        'latency_ms': str(round(latency_ms, 2))
    }
)

# 2. Publish high-severity alert if confidence >= 90%
if confidence >= 0.90:
    sns.publish(
        TopicArn='arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic',
        Subject='[URGENT SECURITY] High-Confidence Phishing Email Detected',
        Message=f"Security alert triggered by AWS Phishing Detection System!\n\n"
                f"Trace Request ID: {request_id}\n"
                f"Phishing Confidence: {confidence * 100:.2f}%\n"
                f"Email Preview: {email_text[:250]}...\n\n"
                f"Immediate administrative review recommended."
    )
```

---

### 4. Technical Challenges & Troubleshooting

* **Issue: `TypeError: Float types are not supported` during DynamoDB put-item operations.**
  * *Symptom:* Invoking `table.put_item()` crashed with `TypeError: Float types are not supported. Use Decimal levels instead`, returning an HTTP 500 error.
  * *Root Cause Analysis:* The Python `boto3` DynamoDB type serializer intentionally rejects native IEEE-754 binary floating-point numbers to prevent silent precision loss.
  * *Resolution:* Converted the float confidence metric to a formatted string `str(round(confidence, 4))` or Python `Decimal` type. Casting to string provided maximum serialization speed and preserved exact JSON representation.

---

### 5. Deliverables & Milestones
1. **Real-time Audit Trail Store:** Provisioned DynamoDB table `PhishingDetectionLogs` delivering sub-10ms write speeds.
2. **Instant Incident Alerting:** Amazon SNS pipeline delivering emergency alerts to the security operations inbox in under 2 seconds.
3. **Least Privilege Enforced:** Clean IAM access boundary strictly limited to single resource ARNs.
