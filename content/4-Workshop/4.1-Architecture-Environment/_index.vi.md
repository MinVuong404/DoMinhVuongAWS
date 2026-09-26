---
title: "Kiến trúc & Môi trường"
date: 2026-08-25
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# 4.1. Tổng quan kiến trúc và chuẩn bị môi trường thực hành

### 4.1.1. Phân tích luồng dữ liệu và thông số kỹ thuật mô hình học máy

Hệ thống phát hiện email lừa đảo được xây dựng theo kiến trúc Serverless trên nền tảng AWS. Luồng xử lý dữ liệu trải qua 5 giai đoạn:

![Sơ đồ Luồng Dữ liệu & Quy trình Suy luận Mô hình AI](/images/architecture/phishing-detection-dataflow.svg)

#### Quy trình xử lý văn bản và suy luận mô hình
1. Tiền xử lý dữ liệu: Bóc tách văn bản từ email, loại bỏ các thẻ HTML style và script, chuẩn hóa liên kết thành chuỗi tagurl, chuẩn hóa địa chỉ email thành tagemail và loại bỏ các ký tự đặc biệt.
2. Biến đổi đặc trưng TF-IDF: Chuyển đổi chuỗi văn bản thành ma trận đặc trưng từ vựng với kích thước từ điển 10.000 unigrams và bigrams.
3. Giảm chiều TruncatedSVD: Nén không gian ma trận từ 10.000 chiều xuống 300 chiều tiềm ẩn, giữ lại trên 92% phương sai ngữ nghĩa và giảm dung lượng bộ nhớ cần thiết trong quá trình suy luận.
4. Dự đoán xác suất bằng XGBoost: Thực hiện phân loại nhị phân tính toán xác suất lừa đảo P nằm trong khoảng 0.0 đến 1.0. Nếu P lớn hơn hoặc bằng 0.5, hệ thống xếp loại là Phishing; ngược lại xếp loại là Safe.
5. Ghi nhận dữ liệu và cảnh báo: Lưu trữ thông số vào DynamoDB và phát cảnh báo qua SNS nếu xác suất P đạt từ 0.90 trở lên.

---

### 4.1.2. Chuẩn bị công cụ và môi trường máy trạm

Trước khi triển khai lên hạ tầng đám mây, môi trường máy trạm cần cài đặt các công cụ sau:

#### 1. Kiểm tra công cụ dòng lệnh AWS CLI v2
```bash
aws --version
```
Yêu cầu đầu ra: Phiên bản aws-cli/2.x.x trở lên.

#### 2. Cấu hình hồ sơ xác thực AWS CLI
```bash
aws configure set default.region ap-southeast-1
aws configure set default.output json

# Xác thực định danh hiện tại
aws sts get-caller-identity
```
Kết quả phản hồi hợp lệ:
```json
{
    "UserId": "AIDA4TESTVUONGFCAJ01",
    "Account": "803146828520",
    "Arn": "arn:aws:iam::803146828520:user/dev-vuong"
}
```

#### 3. Kiểm tra Docker Engine
```bash
docker --version
docker ps
```
Yêu cầu: Docker daemon đang hoạt động ở chế độ nền.

#### 4. Kiểm tra phiên bản Python 3.12
```bash
python --version
```
Yêu cầu: Phiên bản Python 3.12.x phù hợp với môi trường thực thi của Lambda Base Image.

---

### 4.1.3. Thiết lập IAM Execution Role theo đặc quyền tối thiểu

Hàm AWS Lambda cần một IAM Role để được cấp quyền ghi nhật ký vào CloudWatch, ghi dữ liệu vào DynamoDB và gửi thông báo qua SNS.

#### Bước 1: Tạo chính sách tín nhiệm cho phép dịch vụ Lambda đảm nhận vai trò
Tạo tệp trust-policy.json:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### Bước 2: Khởi tạo IAM Role bằng lệnh CLI
```bash
aws iam create-role \
    --role-name PhishingDetectionLambdaRole \
    --assume-role-policy-document file://trust-policy.json \
    --description "IAM Execution Role for Serverless Phishing Detection Lambda Function"
```

#### Bước 3: Gán chính sách ghi nhật ký CloudWatch cơ bản
```bash
aws iam attach-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

#### Bước 4: Lấy mã ARN của IAM Role
```bash
aws iam get-role --role-name PhishingDetectionLambdaRole --query 'Role.Arn' --output text
```
Mã ARN được tạo có định dạng: arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole. Giá trị ARN này được sử dụng trong bài thực hành 4.3.

{{% notice warning %}}
Theo nguyên tắc đặc quyền tối thiểu, bước này chỉ cấp quyền ghi nhật ký cơ bản. Quyền truy cập vào DynamoDB và SNS được cấp bổ sung theo từng ARN tài nguyên trong bài thực hành 4.5 sau khi các tài nguyên đó hoàn tất khởi tạo.
{{% /notice %}}
