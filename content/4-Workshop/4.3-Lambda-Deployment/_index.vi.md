---
title: "Triển khai AWS Lambda"
date: 2026-08-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# 4.3. Triển khai hàm AWS Lambda từ Container Image

### 4.3.1. Khởi tạo hàm AWS Lambda từ Image ECR

Sau khi Container Image được đẩy lên ECR, bước tiếp theo là khởi tạo hàm Lambda mang tên PhishingDetectionXGBoostContainer.

Khởi tạo hàm bằng lệnh AWS CLI:
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
    --description "Serverless Phishing Detection using XGBoost and SVD Container" \
    --region ap-southeast-1
```

Dữ liệu JSON phản hồi:
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

Kiểm tra trạng thái triển khai của hàm cho đến khi giá trị State chuyển sang Active:
```bash
aws lambda get-function \
    --function-name PhishingDetectionXGBoostContainer \
    --query 'Configuration.[State, LastUpdateStatus]' \
    --output text \
    --region ap-southeast-1
```
Kết quả ghi nhận: Active Successful.

---

### 4.3.2. Cấu hình tài nguyên và biến môi trường

| Tham số cấu hình | Giá trị thiết lập | Phân tích lý do kỹ thuật |
| :--- | :--- | :--- |
| Package Type | `Image` | Cho phép sử dụng Docker Image từ ECR, loại bỏ giới hạn 250 MB của tệp Zip. |
| Memory Size | 512 MB | Cung cấp tài nguyên tính toán tương ứng từ hệ thống giúp mô hình XGBoost xử lý dữ liệu ma trận nhanh chóng, dung lượng bộ nhớ thực tế sử dụng ghi nhận 154 MB. |
| Timeout | 5 giây | Đảm bảo thời gian hoàn thành cho lần khởi động ban đầu Cold Start trong khoảng 1.4 đến 1.8 giây, tránh lỗi quá hạn từ API Gateway. |
| Architecture | `x86_64` | Đồng bộ kiến trúc với tệp biên dịch Dockerfile. |
| CONFIDENCE_THRESHOLD | 0.9 | Ngưỡng xác suất kích hoạt thông báo cảnh báo qua Amazon SNS khi phát hiện email lừa đảo có xác suất từ 90% trở lên. |
| DYNAMODB_TABLE | `PhishingDetectionLogs` | Tên bảng NoSQL lưu nhật ký kiểm toán. |
| SNS_TOPIC_ARN | `arn:aws:sns:...` | Mã định danh ARN của chủ đề SNS phát cảnh báo. |

---

### 4.3.3. Tối ưu độ trễ Cold Start bằng kỹ thuật nạp mô hình toàn cục

Mã nguồn lambda_function.py áp dụng kỹ thuật khởi tạo đối tượng mô hình tại phạm vi biến toàn cục bên ngoài hàm xử lý lambda_handler nhằm giảm thời gian nạp tệp ở các lần thực thi tiếp theo:

```python
import json
import os
import re
import time
import uuid
import datetime
import joblib
import boto3

# Nạp mô hình vào bộ nhớ toàn cục một lần khi môi trường microVM khởi tạo
print("--> [INIT] Dang nap mo hinh vao bo nho toan cuc...")
start_init = time.time()

# 1. Khoi tao AWS Boto3 Clients
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

# 2. Doc bien moi truong
TABLE_NAME = os.environ.get('DYNAMODB_TABLE', 'PhishingDetectionLogs')
SNS_TOPIC_ARN = os.environ.get('SNS_TOPIC_ARN', '')
CONFIDENCE_THRESHOLD = float(os.environ.get('CONFIDENCE_THRESHOLD', '0.9'))
table = dynamodb.Table(TABLE_NAME)

# 3. Nap 3 tep trong so mo hinh tu o dia vao RAM
TASK_ROOT = os.environ.get('LAMBDA_TASK_ROOT', '.')
MODEL = joblib.load(os.path.join(TASK_ROOT, 'model.joblib'))
TFIDF = joblib.load(os.path.join(TASK_ROOT, 'tfidf.joblib'))
SVD = joblib.load(os.path.join(TASK_ROOT, 'svd.joblib'))

print(f"--> [INIT] Nap mo hinh hoan tat trong: {(time.time() - start_init)*1000:.2f} ms")


def clean_text(text):
    """Tien xu ly chuoi email van ban tho"""
    text = str(text).lower()
    text = re.sub(r'<[^>]+>', ' ', text)
    text = re.sub(r'https?://\S+|www\.\S+', ' tagurl ', text)
    text = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', ' tagemail ', text)
    text = re.sub(r'[^\w\s]', ' ', text)
    return ' '.join(text.split())


def lambda_handler(event, context):
    """Diem vao xu ly chinh khi co yeu cau tu API Gateway"""
    start_time = time.time()
    request_id = str(uuid.uuid4())
    
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
            "body": json.dumps({"error": "Noi dung email khong duoc de trong"})
        }

    cleaned = clean_text(raw_text)
    tfidf_feat = TFIDF.transform([cleaned])
    svd_feat = SVD.transform(tfidf_feat)
    
    proba = MODEL.predict_proba(svd_feat)[0]
    phishing_prob = float(proba[1])
    is_phishing = phishing_prob >= 0.5
    label = "Phishing" if is_phishing else "Safe"
    risk_level = "CRITICAL" if phishing_prob >= 0.9 else ("HIGH" if is_phishing else "LOW")
    
    latency_ms = (time.time() - start_time) * 1000

    # 1. Ghi vet kiem toan vao DynamoDB
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

    # 2. Phat canh bao qua SNS neu xac suat lua dao vuot nguong
    if is_phishing and phishing_prob >= CONFIDENCE_THRESHOLD and SNS_TOPIC_ARN:
        try:
            sns.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject="Canh bao phat hien email lua dao",
                Message=f"Phat hien email co nguy co lua dao cao.\n\n"
                        f"Request ID: {request_id}\n"
                        f"Muc do rui ro: {risk_level}\n"
                        f"Do tin cay: {phishing_prob*100:.2f}%\n"
                        f"Thoi gian: {datetime.datetime.utcnow().isoformat()}\n"
                        f"Noi dung email: {raw_text[:300]}"
            )
        except Exception as e:
            print(f"SNS Alert Warning: {str(e)}")

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

### 4.3.4. Kiểm thử gọi hàm qua AWS CLI

Tạo tệp dữ liệu kiểm thử payload-test.json:
```json
{
  "body": "{\"email_text\": \"CẢNH BÁO BẢO MẬT: Tài khoản ngân hàng Vietcombank của bạn bị tạm ngưng. Nhấp link https://vietcombank-xac-thuc-otp.xyz ngay lập tức!\"}"
}
```

Thực thi lệnh gọi hàm:
```bash
aws lambda invoke \
    --function-name PhishingDetectionXGBoostContainer \
    --cli-binary-format raw-in-base64-out \
    --payload file://payload-test.json \
    --region ap-southeast-1 \
    response.json
```

Kết quả phản hồi trong tệp response.json:
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

Hàm Lambda phản hồi kết quả phân loại với thời gian suy luận ghi nhận là 12.45 ms.
