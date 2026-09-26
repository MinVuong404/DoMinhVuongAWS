---
title: "AWS Lambda Deployment"
date: 2026-08-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# 4.3. AWS Lambda Function Deployment from Container Image

### 4.3.1. Provisioning AWS Lambda from ECR Container Image

With the container image safely hosted in Amazon ECR, we instantiate the serverless compute function **AWS Lambda** titled `PhishingDetectionXGBoostContainer`.

Execute the following AWS CLI command to provision the function:
```bash
aws lambda create-function \
    --function-name PhishingDetectionXGBoostContainer \
    --package-type Image \
    --code ImageUri=803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest \
    --role arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole \
    --memory-size 512 \
    --timeout 5 \
    --architectures x86_64 \
    --environment "Variables={CONFIDENCE_THRESHOLD=0.9,DYNAMODB_TABLE=PhishingDetectionLogs,SNS_TOPIC_ARN=arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic}" \
    --description "100% Serverless Phishing Detection using XGBoost & SVD Container" \
    --region ap-southeast-1
```

*AWS Lambda JSON response:*
```json
{
    "FunctionName": "PhishingDetectionXGBoostContainer",
    "FunctionArn": "arn:aws:lambda:ap-southeast-1:803146828520:function:PhishingDetectionXGBoostContainer",
    "PackageType": "Image",
    "Role": "arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole",
    "MemorySize": 512,
    "Timeout": 5,
    "State": "Pending",
    "StateReason": "The function is being created.",
    "StateReasonCode": "Creating"
}
```

Poll function readiness until `State` reaches `Active`:
```bash
aws lambda get-function \
    --function-name PhishingDetectionXGBoostContainer \
    --query 'Configuration.[State, LastUpdateStatus]' \
    --output text \
    --region ap-southeast-1
```
*Expected verification output:* `Active Successful`.

---

### 4.3.2. Compute Configuration & Environment Parameter Matrix

| Configuration Parameter | Target Value | Technical Justification |
| :--- | :--- | :--- |
| **Package Type** | `Image` | Ingests OCI Container Images from ECR, breaking past the 250 MB zip barrier. |
| **Memory Size** | **512 MB** | Allocates proportional vCPU compute slices; C-extension matrix calculations finish in milliseconds with ~154 MB peak RAM consumed. |
| **Timeout** | **5 seconds** | Provides adequate safety margin for cold start initialization (1.4 – 1.8s) avoiding premature timeouts. |
| **Architecture** | `x86_64` | Standardized with the `linux/amd64` Docker compilation flag. |
| **CONFIDENCE_THRESHOLD**| `0.9` | Sensitivity threshold dispatching emergency SNS notifications when malicious confidence reaches ≥ 90%. |
| **DYNAMODB_TABLE** | `PhishingDetectionLogs` | Target DynamoDB audit table name. |
| **SNS_TOPIC_ARN** | `arn:aws:sns:...` | Target SNS notification topic ARN. |

---

### 4.3.3. Cold Start Mitigation & Global Scope Pre-loading

To conquer the cold start hurdle, `lambda_function.py` executes heavy machine learning model deserialization **outside** the `lambda_handler` invocation loop:

```python
import json
import os
import re
import time
import uuid
import datetime
import joblib
import boto3

# ====================================================================
# GLOBAL SCOPE OPTIMIZATION (EXECUTED ONCE PER MICROVM CONTAINER LIFECYCLE)
# ====================================================================
print("--> [INIT] Pre-loading Machine Learning models into Global RAM...")
start_init = time.time()

# 1. Initialize AWS SDK Clients
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

# 2. Ingest Environment Variables
TABLE_NAME = os.environ.get('DYNAMODB_TABLE', 'PhishingDetectionLogs')
SNS_TOPIC_ARN = os.environ.get('SNS_TOPIC_ARN', '')
CONFIDENCE_THRESHOLD = float(os.environ.get('CONFIDENCE_THRESHOLD', '0.9'))
table = dynamodb.Table(TABLE_NAME)

# 3. Load ML model weights from disk into shared global memory
TASK_ROOT = os.environ.get('LAMBDA_TASK_ROOT', '.')
MODEL = joblib.load(os.path.join(TASK_ROOT, 'model.joblib'))
TFIDF = joblib.load(os.path.join(TASK_ROOT, 'tfidf.joblib'))
SVD = joblib.load(os.path.join(TASK_ROOT, 'svd.joblib'))

print(f"--> [INIT] Models pre-loaded successfully in: {(time.time() - start_init)*1000:.2f} ms")


def clean_text(text):
    """Sanitize and normalize raw email textual strings"""
    text = str(text).lower()
    text = re.sub(r'<[^>]+>', ' ', text)                          # Strip HTML tags
    text = re.sub(r'https?://\S+|www\.\S+', ' tagurl ', text)     # Normalize URLs
    text = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', ' tagemail ', text) # Normalize emails
    text = re.sub(r'[^\w\s]', ' ', text)                         # Strip special chars
    return ' '.join(text.split())


def lambda_handler(event, context):
    """Main execution entrypoint triggered by API Gateway"""
    start_time = time.time()
    request_id = str(uuid.uuid4())
    
    # Extract email text payload
    try:
        body = event.get('body', {})
        if isinstance(body, str):
            body = json.loads(body)
        raw_text = body.get('email_text', '')
    except Exception:
        raw_text = str(event.get('body', ''))

    if not raw_text.strip():
        return {
            "statusCode": 400,
            "headers": { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" },
            "body": json.dumps({"error": "Email text content cannot be blank!"})
        }

    # Execute ML Inference Pipeline
    cleaned = clean_text(raw_text)
    tfidf_feat = TFIDF.transform([cleaned])
    svd_feat = SVD.transform(tfidf_feat)
    
    proba = MODEL.predict_proba(svd_feat)[0]
    phishing_prob = float(proba[1])
    is_phishing = phishing_prob >= 0.5
    label = "Phishing" if is_phishing else "Safe"
    risk_level = "CRITICAL" if phishing_prob >= 0.9 else ("HIGH" if is_phishing else "LOW")
    
    latency_ms = (time.time() - start_time) * 1000

    # 1. Persist audit record in DynamoDB (Gracefully catch unprovisioned state)
    try:
        table.put_item(
            Item={
                'request_id': request_id,
                'timestamp': datetime.datetime.utcnow().isoformat(),
                'prediction': label,
                'confidence': str(round(phishing_prob, 4)),
                'risk_level': risk_level,
                'text_preview': raw_text[:250],
                'latency_ms': str(round(latency_ms, 2))
            }
        )
    except Exception as e:
        print(f"DynamoDB Log Warning: {str(e)}")

    # 2. Publish emergency SNS alert if risk >= 90%
    if is_phishing and phishing_prob >= CONFIDENCE_THRESHOLD and SNS_TOPIC_ARN:
        try:
            sns.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject=f"[URGENT SECURITY] High-Confidence Phishing Email Detected ({phishing_prob*100:.1f}%)",
                Message=f"Security alert triggered by AWS Phishing Detection System!\n\n"
                        f"Trace Request ID: {request_id}\n"
                        f"Risk Severity: {risk_level}\n"
                        f"Phishing Confidence: {phishing_prob*100:.2f}%\n"
                        f"Timestamp: {datetime.datetime.utcnow().isoformat()}\n"
                        f"Email Snippet: {raw_text[:300]}..."
            )
        except Exception as e:
            print(f"SNS Alert Warning: {str(e)}")

    # Return response payload to API Gateway and Client
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "POST,OPTIONS"
        },
        "body": json.dumps({
            "request_id": request_id,
            "prediction": label,
            "confidence": round(phishing_prob, 4),
            "confidence_percent": f"{phishing_prob*100:.2f}%",
            "risk_level": risk_level,
            "inference_latency_ms": round(latency_ms, 2)
        })
    }
```

---

### 4.3.4. Direct Invocation Testing via AWS CLI

Create test fixture payload `payload-test.json`:
```json
{
  "body": "{\"email_text\": \"SECURITY ALERT: Your online banking account has been suspended. Click https://bank-verify-auth.xyz immediately!\"}"
}
```

Execute direct Lambda invocation:
```bash
aws lambda invoke \
    --function-name PhishingDetectionXGBoostContainer \
    --cli-binary-format raw-in-base64-out \
    --payload file://payload-test.json \
    --region ap-southeast-1 \
    response.json
```

Inspect output stored in `response.json`:
```json
{
    "statusCode": 200,
    "headers": {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "POST,OPTIONS"
    },
    "body": "{\"request_id\": \"d73e91a2-8b9a-4f51-92cb-5079a0cf5823\", \"prediction\": \"Phishing\", \"confidence\": 0.9854, \"confidence_percent\": \"98.54%\", \"risk_level\": \"CRITICAL\", \"inference_latency_ms\": 12.45}"
}
```

The function reliably categorized the payload with an internal compute latency of only **12.45 ms**.
