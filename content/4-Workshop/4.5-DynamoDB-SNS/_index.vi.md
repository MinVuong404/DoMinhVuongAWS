---
title: "DynamoDB & Amazon SNS"
date: 2026-08-25
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# 4.5. Tích hợp lưu trữ Amazon DynamoDB và cảnh báo Amazon SNS

### 4.5.1. Khởi tạo bảng NoSQL Amazon DynamoDB

Dịch vụ Amazon DynamoDB lưu trữ nhật ký kiểm toán cho toàn bộ yêu cầu phân tích thư của người dùng, hỗ trợ tra cứu lịch sử sự cố và cung cấp dữ liệu huấn luyện lại mô hình sau này.

Khởi tạo bảng PhishingDetectionLogs với thông lượng 1 RCU và 1 WCU theo chính sách AWS Free Tier:
```bash
aws dynamodb create-table \
    --table-name PhishingDetectionLogs \
    --attribute-definitions AttributeName=request_id,AttributeType=S \
    --key-schema AttributeName=request_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region ap-southeast-1
```

Kiểm tra trạng thái bảng cho đến khi giá trị chuyển sang ACTIVE:
```bash
aws dynamodb describe-table \
    --table-name PhishingDetectionLogs \
    --query 'Table.[TableStatus, TableArn]' \
    --output text \
    --region ap-southeast-1
```
Ghi nhận mã ARN của bảng: arn:aws:dynamodb:ap-southeast-1:803146828520:table/PhishingDetectionLogs.

---

### 4.5.2. Khởi tạo Amazon SNS Topic và đăng ký email cảnh báo

Dịch vụ Amazon Simple Notification Service hoạt động theo kiến trúc gửi nhận bất đồng bộ, giúp gửi thông báo tới quản trị viên mà không ảnh hưởng tới thời gian phản hồi của API.

#### Bước 1: Tạo SNS Topic
```bash
aws sns create-topic \
    --name PhishingAlertTopic \
    --region ap-southeast-1
```
Ghi nhận mã ARN của chủ đề: arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic.

#### Bước 2: Đăng ký email nhận thông báo sự cố
```bash
aws sns subscribe \
    --topic-arn arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region ap-southeast-1
```

{{% notice warning %}}
Truy cập hòm thư vuong.dmforwork@gmail.com và nhấp vào liên kết Confirm subscription trong thư thông báo của AWS để kích hoạt kênh nhận cảnh báo.
{{% /notice %}}

---

### 4.5.3. Cấp quyền truy cập cho Lambda Execution Role

Tạo tệp chính sách lambda-dynamodb-sns-policy.json:
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

Gán chính sách nội tuyến vào vai trò PhishingDetectionLambdaRole:
```bash
aws iam put-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-name DynamoDBAndSNSPolicy \
    --policy-document file://lambda-dynamodb-sns-policy.json
```

---

### 4.5.4. Kiểm thử kích hoạt cảnh báo và kiểm tra dữ liệu trong DynamoDB

Gửi mẫu thư lừa đảo kiểm thử qua API Gateway:
```bash
curl -X POST https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
    -H "Content-Type: application/json" \
    -d '{
        "email_text": "CẢNH BÁO BẢO MẬT KHẨN CẤP: Tài khoản Techcombank của bạn bị phát hiện đăng nhập bất thường tại Nga. Vui lòng bấm vào https://techcombank-security-otp.cc để hủy lệnh chuyển tiền 50.000.000 VNĐ ngay bây giờ!"
    }'
```

Dữ liệu JSON phản hồi:
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

#### 1. Kiểm tra bản ghi trong bảng DynamoDB
```bash
aws dynamodb get-item \
    --table-name PhishingDetectionLogs \
    --key '{"request_id": {"S": "a4f89d12-612b-4e01-9a73-8cb23e41b0fa"}}' \
    --region ap-southeast-1 \
    --output json
```

Dữ liệu bản ghi kiểm toán ghi nhận:
```json
{
    "Item": {
        "request_id": { "S": "a4f89d12-612b-4e01-9a73-8cb23e41b0fa" },
        "prediction": { "S": "Phishing" },
        "confidence": { "S": "0.9912" },
        "risk_level": { "S": "CRITICAL" },
        "latency_ms": { "S": "13.12" },
        "timestamp": { "S": "2026-08-25T14:30:15.120Z" },
        "text_preview": { "S": "CẢNH BÁO BẢO MẬT KHẨN CẤP: Tài khoản Techcombank của bạn bị phát hiện đăng nhập bất thường..." }
    }
}
```

#### 2. Kiểm tra email cảnh báo từ Amazon SNS
Sau 1.5 giây, hòm thư vuong.dmforwork@gmail.com nhận được email thông báo:
* Tiêu đề: Canh bao phat hien email lua dao
* Nội dung: Thông tin mã yêu cầu, xác suất lừa đảo 99.12%, mức độ rủi ro CRITICAL và đoạn trích nội dung email.
