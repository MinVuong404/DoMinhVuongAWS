---
title: "Amazon API Gateway v2"
date: 2026-08-25
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# 4.4. Amazon API Gateway v2 Ingestion Ingress Setup

### 4.4.1. HTTP API v2 Instantiation & Route Configuration

To allow the browser extension to communicate securely across public Internet networks via HTTPS, we configure **Amazon API Gateway v2** utilizing the modern **HTTP API** standard. This protocol delivers 71% cost savings and a 60% reduction in networking overhead compared to legacy REST APIs.

#### Step 1: Instantiate HTTP API with Native CORS
```bash
aws apigatewayv2 create-api \
    --name PhishingDetectionHttpApi \
    --protocol-type HTTP \
    --cors-configuration "AllowOrigins=[\"*\"],AllowMethods=[\"POST\",\"OPTIONS\"],AllowHeaders=[\"Content-Type\",\"Authorization\"],MaxAge=300" \
    --region ap-southeast-1
```
*Capture returned `ApiId`:* e.g., `p7ailap9ci`.

#### Step 2: Create Integration Wiring to AWS Lambda
```bash
aws apigatewayv2 create-integration \
    --api-id p7ailap9ci \
    --integration-type AWS_PROXY \
    --integration-uri arn:aws:lambda:ap-southeast-1:803146828520:function:PhishingDetectionXGBoostContainer \
    --payload-format-version 2.0 \
    --region ap-southeast-1
```
*Capture returned `IntegrationId`:* e.g., `int-9a8b7c6d`.

#### Step 3: Configure Ingestion Route `POST /predict`
```bash
aws apigatewayv2 create-route \
    --api-id p7ailap9ci \
    --route-key "POST /predict" \
    --target "integrations/int-9a8b7c6d" \
    --region ap-southeast-1
```

#### Step 4: Provision Default Stage with Auto-Deploy
```bash
aws apigatewayv2 create-stage \
    --api-id p7ailap9ci \
    --stage-name '$default' \
    --auto-deploy \
    --region ap-southeast-1
```

---

### 4.4.2. Authorizing API Gateway Lambda Invocations

Under AWS Zero-Trust governance, API Gateway possesses zero inherent permissions to trigger AWS Lambda without explicit **Resource-based Policies**.

Execute the following permission grant:
```bash
aws lambda add-permission \
    --function-name PhishingDetectionXGBoostContainer \
    --statement-id AllowApiGatewayInvoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:ap-southeast-1:803146828520:p7ailap9ci/*/*/predict" \
    --region ap-southeast-1
```

*Policy statement verification:*
```json
{
    "Statement": "{\"Sid\":\"AllowApiGatewayInvoke\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:ap-southeast-1:803146828520:function:PhishingDetectionXGBoostContainer\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:ap-southeast-1:803146828520:p7ailap9ci/*/*/predict\"}}}"
}
```

---

### 4.4.3. Cross-Origin Resource Sharing (CORS) Verification

Because the client extension executes inside the context of `https://mail.google.com`, browser security prevents cross-origin data fetching unless explicit **CORS** headers are confirmed.

Verify API Gateway CORS parameters:
```bash
aws apigatewayv2 get-api \
    --api-id p7ailap9ci \
    --query 'CorsConfiguration' \
    --output json \
    --region ap-southeast-1
```

*Expected CORS configuration:*
```json
{
    "AllowHeaders": [
        "content-type",
        "authorization"
    ],
    "AllowMethods": [
        "POST",
        "OPTIONS"
    ],
    "AllowOrigins": [
        "*"
    ],
    "MaxAge": 300
}
```

---

### 4.4.4. Public Endpoint Validation via cURL

Issue a live HTTPS POST request to validate the operational gateway:
```bash
curl -X POST https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
    -H "Content-Type: application/json" \
    -d '{
        "email_text": "Dear Valued Customer, Vietcombank confirms successful transfer transaction reference 982734 into your account."
    }'
```

*Instantaneous JSON response:*
```json
{
    "request_id": "c1a9f04e-4f51-4091-884d-2a1c720e36b8",
    "prediction": "Safe",
    "confidence": 0.0412,
    "confidence_percent": "4.12%",
    "risk_level": "LOW",
    "inference_latency_ms": 11.85
}
```

The public HTTPS gateway endpoint is fully operational and ready for Chrome Extension client traffic.
