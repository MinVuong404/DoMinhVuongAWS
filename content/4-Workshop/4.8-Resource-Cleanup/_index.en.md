---
title: "Resource Clean-up"
date: 2026-08-25
weight: 8
chapter: false
pre: " <b> 4.8. </b> "
---

# 4.8. Resource Clean-up Guide

### 4.8.1. Significance of Cloud Resource Deprovisioning

In alignment with the **AWS Well-Architected Cost Optimization Pillar**, releasing cloud infrastructure upon completion of validation testing or conclusion of an internship sprint is essential to:
1. Prevent unintended financial liabilities once promotional tiers or Free Tier allowances lapse.
2. Maintain clean, auditable cloud environments complying with Cloud Resource Lifecycle Management standards.

---

### 4.8.2. Visual Teardown via AWS Management Console

To manually release provisioned resources via the graphical user interface, execute the teardown in reverse dependency order:

1. **Amazon API Gateway:** Open API Gateway Console → Select `PhishingDetectionHttpApi` → Click **Actions** → Select **Delete**.
2. **AWS Lambda:** Open Lambda Console → Select `PhishingDetectionXGBoostContainer` → Click **Actions** → Select **Delete function**.
3. **Amazon ECR:** Open ECR Console → Select repository `phishing-xgboost` → Click **Delete** → Enter `delete` to confirm removal of all container images.
4. **Amazon DynamoDB:** Open DynamoDB Console → Select table `PhishingDetectionLogs` → Click **Delete table** → Confirm.
5. **Amazon SNS:** Open SNS Console → Select Topic `PhishingAlertTopic` → Click **Delete**. Remove associated subscriptions.
6. **Amazon CloudWatch:** 
   * Open **Dashboards** → Select `PhishingDetection-Operations-Dashboard` → Click **Delete**.
   * Open **Alarms** → Select `Lambda-Execution-Errors-Alarm` and `Lambda-High-Duration-Alarm` → Click **Delete**.
   * Open **Log groups** → Select `/aws/lambda/PhishingDetectionXGBoostContainer` → Click **Delete log group**.
7. **AWS IAM:** Open IAM Console → Roles → Select `PhishingDetectionLambdaRole` → Delete inline policy `DynamoDBAndSNSPolicy` → Detach managed policy → Click **Delete Role**.

---

### 4.8.3. Comprehensive Automated Teardown Script via AWS CLI

To ensure comprehensive deletion and avoid orphan resources, execute the following automated Bash / PowerShell script:

```bash
#!/bin/bash
REGION="ap-southeast-1"
ACCOUNT_ID="803146828520"

echo "=== COMMENCING TEARDOWN: SERVERLESS PHISHING DETECTION INFRASTRUCTURE ==="

# 1. Purge CloudWatch Alarms & Operational Dashboard
echo "--> Deleting CloudWatch Alarms & Dashboard..."
aws cloudwatch delete-alarms \
    --alarm-names "Lambda-Execution-Errors-Alarm" "Lambda-High-Duration-Alarm" \
    --region $REGION
aws cloudwatch delete-dashboards \
    --dashboard-names "PhishingDetection-Operations-Dashboard" \
    --region $REGION

# 2. Purge CloudWatch Log Group
echo "--> Deleting CloudWatch Log Group..."
aws logs delete-log-group \
    --log-group-name "/aws/lambda/PhishingDetectionXGBoostContainer" \
    --region $REGION 2>/dev/null || true

# 3. Purge Amazon SNS Topic
echo "--> Deleting Amazon SNS Topic..."
aws sns delete-topic \
    --topic-arn "arn:aws:sns:${REGION}:${ACCOUNT_ID}:PhishingAlertTopic" \
    --region $REGION

# 4. Purge Amazon DynamoDB NoSQL Table
echo "--> Deleting Amazon DynamoDB Table..."
aws dynamodb delete-table \
    --table-name "PhishingDetectionLogs" \
    --region $REGION

# 5. Purge Amazon API Gateway v2
echo "--> Querying and deleting Amazon API Gateway v2..."
API_ID=$(aws apigatewayv2 get-apis --region $REGION --query "Items[?Name=='PhishingDetectionHttpApi'].ApiId" --output text)
if [ -n "$API_ID" ] && [ "$API_ID" != "None" ]; then
    aws apigatewayv2 delete-api --api-id $API_ID --region $REGION
    echo "    Deleted API Gateway ID: $API_ID"
fi

# 6. Purge AWS Lambda Function
echo "--> Deleting AWS Lambda Function..."
aws lambda delete-function \
    --function-name "PhishingDetectionXGBoostContainer" \
    --region $REGION

# 7. Force-delete Amazon ECR Repository and All Images
echo "--> Deleting Amazon ECR Repository..."
aws ecr delete-repository \
    --repository-name "phishing-xgboost" \
    --force \
    --region $REGION

# 8. Delete IAM Role and Attached Policies
echo "--> Deleting IAM Role & Policies..."
aws iam delete-role-policy \
    --role-name "PhishingDetectionLambdaRole" \
    --policy-name "DynamoDBAndSNSPolicy" 2>/dev/null || true

aws iam detach-role-policy \
    --role-name "PhishingDetectionLambdaRole" \
    --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole" 2>/dev/null || true

aws iam delete-role \
    --role-name "PhishingDetectionLambdaRole"

echo "=== TEARDOWN COMPLETED SUCCESSFULLY! ONGOING LIABILITY = $0.00 ==="
```

With the execution of this routine, all cloud resources are permanently released, ensuring zero ongoing billing exposure.
