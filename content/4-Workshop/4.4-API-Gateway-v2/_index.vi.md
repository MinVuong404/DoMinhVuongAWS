---
title: "Amazon API Gateway v2"
date: 2026-08-25
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# 4.4. Cấu hình cổng kết nối Amazon API Gateway v2

### 4.4.1. Khởi tạo HTTP API Gateway v2 và cấu hình tuyến đường

Để tiện ích mở rộng Chrome Extension giao tiếp qua giao thức HTTPS công khai, hệ thống khởi tạo cổng Amazon API Gateway v2 chuẩn HTTP API. Chuẩn này giảm 71% chi phí và giảm hơn 60% độ trễ mạng so với REST API.

#### Bước 1: Khởi tạo HTTP API
```bash
aws apigatewayv2 create-api \
    --name PhishingDetectionHttpApi \
    --protocol-type HTTP \
    --cors-configuration "AllowOrigins=[\"*\"],AllowMethods=[\"POST\",\"OPTIONS\"],AllowHeaders=[\"Content-Type\",\"Authorization\"],MaxAge=300" \
    --region ap-southeast-1
```
Ghi nhận giá trị ApiId tạo ra, ví dụ p7ailap9ci.

#### Bước 2: Tạo tích hợp kết nối với hàm AWS Lambda
```bash
aws apigatewayv2 create-integration \
    --api-id p7ailap9ci \
    --integration-type AWS_PROXY \
    --integration-uri arn:aws:lambda:ap-southeast-1:803146828520:function:PhishingDetectionXGBoostContainer \
    --payload-format-version 2.0 \
    --region ap-southeast-1
```
Ghi nhận giá trị IntegrationId tạo ra, ví dụ int-9a8b7c6d.

#### Bước 3: Tạo tuyến đường định tuyến POST /predict
```bash
aws apigatewayv2 create-route \
    --api-id p7ailap9ci \
    --route-key "POST /predict" \
    --target "integrations/int-9a8b7c6d" \
    --region ap-southeast-1
```

#### Bước 4: Khởi tạo giai đoạn triển khai mặc định
```bash
aws apigatewayv2 create-stage \
    --api-id p7ailap9ci \
    --stage-name '$default' \
    --auto-deploy \
    --region ap-southeast-1
```

---

### 4.4.2. Cấp quyền cho API Gateway thực thi hàm Lambda

Theo chính sách bảo mật của AWS, API Gateway cần được cấp quyền gọi hàm rõ ràng thông qua Lambda Resource-based Policy.

Thực thi lệnh cấp quyền:
```bash
aws lambda add-permission \
    --function-name PhishingDetectionXGBoostContainer \
    --statement-id AllowApiGatewayInvoke \
    --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com \
    --source-arn "arn:aws:execute-api:ap-southeast-1:803146828520:p7ailap9ci/*/*/predict" \
    --region ap-southeast-1
```

Dữ liệu JSON phản hồi xác nhận quyền thực thi:
```json
{
    "Statement": "{\"Sid\":\"AllowApiGatewayInvoke\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:ap-southeast-1:803146828520:function:PhishingDetectionXGBoostContainer\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:ap-southeast-1:803146828520:p7ailap9ci/*/*/predict\"}}}"
}
```

---

### 4.4.3. Cấu hình cơ chế chia sẻ tài nguyên CORS cho trình duyệt

Khi Chrome Extension chạy trên trang mail.google.com, trình duyệt sẽ chặn các yêu cầu nếu máy chủ không khai báo tiêu đề Cross-Origin Resource Sharing.

Kiểm tra cấu hình CORS của API Gateway:
```bash
aws apigatewayv2 get-api \
    --api-id p7ailap9ci \
    --query 'CorsConfiguration' \
    --output json \
    --region ap-southeast-1
```

Cấu hình trả về:
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

### 4.4.4. Kiểm thử điểm cuối API qua cURL

Sử dụng lệnh curl gửi yêu cầu POST đến địa chỉ API công khai:
```bash
curl -X POST https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
    -H "Content-Type: application/json" \
    -d '{
        "email_text": "Kính gửi quý khách, Vietcombank xin trân trọng thông báo giao dịch chuyển tiền số 982734 thành công vào tài khoản của quý khách."
    }'
```

Kết quả phản hồi từ hệ thống:
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

Điểm cuối HTTPS sẵn sàng tiếp nhận yêu cầu phân tích từ Chrome Extension.
