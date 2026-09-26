---
title: "Dọn dẹp tài nguyên"
date: 2026-08-25
weight: 8
chapter: false
pre: " <b> 4.8. </b> "
---

# 4.8. Dọn dẹp tài nguyên

### 4.8.1. Mục đích của việc dọn dẹp tài nguyên

Theo khuyến nghị tối ưu chi phí của AWS Well-Architected Framework, việc giải phóng tài nguyên sau khi kết thúc chu kỳ thực nghiệm là cần thiết nhằm:
1. Ngăn chặn phát sinh chi phí khi tài khoản hết thời hạn ưu đãi miễn phí.
2. Quản lý vòng đời tài nguyên và duy trì cấu hình tài khoản gọn gàng.

---

### 4.8.2. Hướng dẫn dọn dẹp qua AWS Management Console

Thực hiện thao tác xóa thủ công trên giao diện quản trị theo thứ tự:

1. Amazon API Gateway: Mở giao diện API Gateway, chọn PhishingDetectionHttpApi, nhấp Actions và chọn Delete.
2. AWS Lambda: Mở giao diện Lambda, chọn hàm PhishingDetectionXGBoostContainer, nhấp Actions và chọn Delete function.
3. Amazon ECR: Mở giao diện ECR, chọn kho lưu trữ phishing-xgboost, nhấp Delete và nhập delete để xác nhận xóa các container images.
4. Amazon DynamoDB: Mở giao diện DynamoDB, chọn bảng PhishingDetectionLogs, nhấp Delete table và xác nhận xóa.
5. Amazon SNS: Mở giao diện SNS, chọn Topic PhishingAlertTopic, nhấp Delete và xóa các Subscriptions liên kết.
6. Amazon CloudWatch:
   * Vào mục Dashboards, chọn PhishingDetection-Operations-Dashboard và nhấp Delete.
   * Vào mục Alarms, chọn Lambda-Execution-Errors-Alarm và Lambda-High-Duration-Alarm, nhấp Delete.
   * Vào mục Log groups, chọn /aws/lambda/PhishingDetectionXGBoostContainer và nhấp Delete log group.
7. AWS IAM: Mở giao diện IAM, chọn Roles, chọn PhishingDetectionLambdaRole, xóa Inline Policy DynamoDBAndSNSPolicy, tách Managed Policy và nhấp Delete Role.

---

### 4.8.3. Kịch bản dọn dẹp tài nguyên tự động bằng AWS CLI

Tập lệnh dọn dẹp tuần tự các tài nguyên đã tạo trong dự án:

```bash
#!/bin/bash
REGION="ap-southeast-1"
ACCOUNT_ID="803146828520"

echo "=== BAT DAU QUY TRINH DON DEP TAI NGUYEN ==="

# 1. Xoa CloudWatch Alarms va Dashboard
echo "--> Xoa CloudWatch Alarms va Dashboard..."
aws cloudwatch delete-alarms \
    --alarm-names "Lambda-Execution-Errors-Alarm" "Lambda-High-Duration-Alarm" \
    --region $REGION
aws cloudwatch delete-dashboards \
    --dashboard-names "PhishingDetection-Operations-Dashboard" \
    --region $REGION

# 2. Xoa CloudWatch Log Group
echo "--> Xoa CloudWatch Log Group..."
aws logs delete-log-group \
    --log-group-name "/aws/lambda/PhishingDetectionXGBoostContainer" \
    --region $REGION 2>/dev/null || true

# 3. Xoa Amazon SNS Topic
echo "--> Xoa Amazon SNS Topic..."
aws sns delete-topic \
    --topic-arn "arn:aws:sns:${REGION}:${ACCOUNT_ID}:PhishingAlertTopic" \
    --region $REGION

# 4. Xoa bang Amazon DynamoDB
echo "--> Xoa bang Amazon DynamoDB..."
aws dynamodb delete-table \
    --table-name "PhishingDetectionLogs" \
    --region $REGION

# 5. Xoa Amazon API Gateway v2
echo "--> Tim va xoa Amazon API Gateway v2..."
API_ID=$(aws apigatewayv2 get-apis --region $REGION --query "Items[?Name=='PhishingDetectionHttpApi'].ApiId" --output text)
if [ -n "$API_ID" ] && [ "$API_ID" != "None" ]; then
    aws apigatewayv2 delete-api --api-id $API_ID --region $REGION
    echo "    Da xoa API Gateway ID: $API_ID"
fi

# 6. Xoa ham AWS Lambda
echo "--> Xoa ham AWS Lambda..."
aws lambda delete-function \
    --function-name "PhishingDetectionXGBoostContainer" \
    --region $REGION

# 7. Xoa kho luu tru Amazon ECR
echo "--> Xoa Amazon ECR Repository..."
aws ecr delete-repository \
    --repository-name "phishing-xgboost" \
    --force \
    --region $REGION

# 8. Xoa IAM Role va cac chinh sach dinh kem
echo "--> Xoa IAM Role va Policies..."
aws iam delete-role-policy \
    --role-name "PhishingDetectionLambdaRole" \
    --policy-name "DynamoDBAndSNSPolicy" 2>/dev/null || true

aws iam detach-role-policy \
    --role-name "PhishingDetectionLambdaRole" \
    --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole" 2>/dev/null || true

aws iam delete-role \
    --role-name "PhishingDetectionLambdaRole"

echo "=== QUY TRINH DON DEP HOAN TAT ==="
```

Sau khi chạy tập lệnh, các tài nguyên khởi tạo trong bài thực hành được giải phóng khỏi tài khoản AWS.
