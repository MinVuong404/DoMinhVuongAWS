---
title: "Nhật ký Tuần 6"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Nhật ký Tuần 6: Tích hợp cơ sở dữ liệu Amazon DynamoDB và hệ thống cảnh báo Amazon SNS

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 07/09/2026 đến 13/09/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Thiết kế cấu trúc dữ liệu NoSQL cho bảng Amazon DynamoDB PhishingDetectionLogs để ghi nhận nhật ký kiểm toán theo thời gian thực.
  2. Thiết lập chế độ thông lượng Provisioned với 1 RCU và 1 WCU nhằm tận dụng chính sách AWS Free Tier với 25 GB dung lượng lưu trữ miễn phí.
  3. Khởi tạo chủ đề Amazon SNS PhishingAlertTopic và cấu hình kênh gửi cảnh báo qua giao thức Email.
  4. Cập nhật chính sách truy cập IAM cấp các quyền dynamodb:PutItem và sns:Publish theo nguyên tắc đặc quyền tối thiểu.
  5. Xây dựng logic cảnh báo tự động: Khi phát hiện email lừa đảo có độ tin cậy từ 90% trở lên, hệ thống tự động phát thông báo tới email quản trị viên.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Thiết kế cấu trúc bảng NoSQL với khóa phân vùng Partition Key là request_id kiểu chuỗi UUIDv4. Xác định các trường dữ liệu gồm timestamp, text_preview, prediction, confidence, risk_level và latency_ms. | 07/09/2026 | 07/09/2026 | [Amazon DynamoDB Core Components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html) | Hoàn thành bản thiết kế cấu trúc dữ liệu cho bảng lưu trữ kiểm toán. |
| Thứ 3 | Tạo bảng PhishingDetectionLogs bằng công cụ dòng lệnh AWS CLI. Cấu hình thông lượng 1 RCU và 1 WCU, kích hoạt tính năng mã hóa dữ liệu KMS tại chỗ. | 08/09/2026 | 08/09/2026 | [AWS CLI DynamoDB create-table](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/dynamodb/create-table.html) | Bảng chuyển sang trạng thái hoạt động Active sau 12 giây, không phát sinh chi phí duy trì. |
| Thứ 4 | Tạo chủ đề Amazon SNS PhishingAlertTopic tại khu vực ap-southeast-1. Đăng ký địa chỉ email vuong.dmforwork@gmail.com vào chủ đề và xác nhận đường dẫn kích hoạt trong hòm thư nhận. | 09/09/2026 | 09/09/2026 | [Amazon SNS Getting Started](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html) | Trạng thái đăng ký chuyển sang mức xác nhận Confirmed với mã ARN hợp lệ. |
| Thứ 5 | Soạn thảo chính sách IAM Policy cấp quyền thực thi tối thiểu: Cho phép hàm Lambda thực hiện lệnh dynamodb:PutItem tới bảng PhishingDetectionLogs và lệnh sns:Publish tới chủ đề PhishingAlertTopic. Gán chính sách vào vai trò PhishingDetectionLambdaRole. | 10/09/2026 | 10/09/2026 | [IAM Policies for Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-identity-based.html) | Phân quyền truy cập chính xác tới các tài nguyên được chỉ định. |
| Thứ 6 | Cập nhật mã nguồn lambda_function.py, tích hợp thư viện boto3 để ghi dữ liệu vào DynamoDB và phát cảnh báo qua SNS. Soạn thảo định dạng nội dung thư cảnh báo gồm mã định danh yêu cầu, xác suất và đoạn trích nội dung. | 11/09/2026 | 11/09/2026 | [Boto3 DynamoDB Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html)<br>[Boto3 SNS Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html) | Mã nguồn ghi nhận dữ liệu và gửi cảnh báo hoạt động ổn định. |
| Thứ 7 và Chủ nhật | Kiểm thử kịch bản email giả mạo với mẫu thư có độ tin cậy 98.4%. Bản ghi được lưu vào DynamoDB trong 8 ms và hệ thống gửi email cảnh báo về hòm thư quản trị viên sau 1.2 giây. | 12/09/2026 | 13/09/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) | Quy trình lưu trữ vết kiểm toán và phát cảnh báo hoàn tất kiểm thử thực tế. |

---

### 3. Thao tác kỹ thuật và mã lệnh cấu hình thực tế

#### 3.1. Lệnh tạo bảng DynamoDB và SNS Topic
```bash
# 1. Khởi tạo bảng DynamoDB theo chuẩn Free Tier
aws dynamodb create-table \
    --table-name PhishingDetectionLogs \
    --attribute-definitions AttributeName=request_id,AttributeType=S \
    --key-schema AttributeName=request_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region ap-southeast-1

# 2. Khởi tạo SNS Topic phát cảnh báo sự cố
aws sns create-topic \
    --name PhishingAlertTopic \
    --region ap-southeast-1

# 3. Đăng ký email quản trị nhận thông báo
aws sns subscribe \
    --topic-arn arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region ap-southeast-1
```

#### 3.2. Chính sách IAM cấp quyền truy cập cho Lambda
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

#### 3.3. Đoạn mã Python xử lý lưu nhật ký và gửi cảnh báo
```python
import boto3
import json
import uuid
import datetime

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')
table = dynamodb.Table('PhishingDetectionLogs')

# 1. Ghi vết kiểm toán vào DynamoDB
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

# 2. Phát cảnh báo nếu độ tin cậy lừa đảo từ 90% trở lên
if confidence >= 0.90:
    sns.publish(
        TopicArn='arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic',
        Subject='[CẢNH BÁO] Phát hiện email lừa đảo',
        Message=f"Hệ thống phát hiện email có nguy cơ lừa đảo cao.\n\n"
                f"Mã định danh yêu cầu: {request_id}\n"
                f"Xác suất lừa đảo: {confidence * 100:.2f}%\n"
                f"Nội dung trích đoạn: {email_text[:250]}...\n\n"
                f"Kiểm tra và xử lý theo quy trình."
    )
```

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Lỗi TypeError khi ghi giá trị số thực float vào DynamoDB.
  * Hiện tượng: Khi hàm Lambda thực thi table.put_item, hệ thống trả về ngoại lệ TypeError thông báo không hỗ trợ kiểu dữ liệu Float và yêu cầu sử dụng Decimal.
  * Nguyên nhân: Thư viện boto3 khi giao tiếp với DynamoDB không hỗ trợ trực tiếp kiểu dữ liệu số thực tiêu chuẩn của Python nhằm tránh sai số làm tròn số học nhị phân.
  * Giải pháp xử lý: Chuyển đổi giá trị số thực sang kiểu Decimal hoặc chuyển đổi thành chuỗi ký tự bằng hàm str round confidence, 4 trước khi gửi bản ghi vào DynamoDB. Phương án chuyển sang chuỗi ký tự giúp định dạng dữ liệu đồng nhất và dễ dàng hiển thị trong các báo cáo kiểm toán.

---

### 5. Kết quả hoàn thành

1. Khởi tạo bảng DynamoDB PhishingDetectionLogs phục vụ lưu trữ nhật ký kiểm toán với thời gian phản hồi ghi dưới 10 ms.
2. Thiết lập chủ đề Amazon SNS gửi email cảnh báo tự động tới ban quản trị với thời gian chuyển tiếp dưới 2 giây.
3. Cấu hình chính sách IAM kiểm soát chi tiết quyền truy cập trên từng ARN tài nguyên theo chuẩn đặc quyền tối thiểu.
