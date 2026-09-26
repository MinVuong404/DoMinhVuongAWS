---
title: "Nhật ký Tuần 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Nhật ký Tuần 5: Triển khai hàm AWS Lambda Container và tối ưu độ trễ suy luận

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 31/08/2026 đến 06/09/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Tạo IAM Execution Role PhishingDetectionLambdaRole theo nguyên tắc đặc quyền tối thiểu.
  2. Khởi tạo hàm AWS Lambda PhishingDetectionXGBoostContainer từ Container Image trên Amazon ECR.
  3. Đo lường hiệu năng trên các mức cấp phát bộ nhớ 128 MB, 256 MB, 512 MB và 1024 MB để xác định cấu hình phù hợp giữa chi phí và tốc độ xử lý.
  4. Áp dụng kỹ thuật nạp mô hình vào bộ nhớ đệm toàn cục bên ngoài phạm vi hàm xử lý lambda_handler nhằm giảm thời gian nạp tệp lặp lại.
  5. Thiết lập thời gian chờ 5 giây và cấu hình các biến môi trường gồm CONFIDENCE_THRESHOLD, DYNAMODB_TABLE và SNS_TOPIC_ARN.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Tạo chính sách tín nhiệm cho phép dịch vụ Lambda đảm nhận vai trò thực thi. Gán quyền cơ bản AWSLambdaBasicExecutionRole để ghi nhật ký vào CloudWatch. | 31/08/2026 | 31/08/2026 | [AWS Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html) | Tạo thành công IAM Role có mã định danh arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole. |
| Thứ 3 | Khởi tạo hàm Lambda bằng lệnh aws lambda create-function, chỉ định kiểu đóng gói Image trỏ tới địa chỉ Container Image trên ECR. Kiểm tra trạng thái triển khai chuyển sang mức Active. | 01/09/2026 | 01/09/2026 | [AWS CLI Lambda create-function](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/lambda/create-function.html) | Hàm Lambda được khởi tạo thành công tại khu vực Singapore ap-southeast-1. |
| Thứ 4 | Kiểm thử gọi hàm với dữ liệu mẫu để đo lường độ trễ khởi động ban đầu. Lần gọi đầu tiên mất 1420 ms do hệ thống tải và giải nén Container Image. Phân tích nhật ký thực thi qua chỉ số REPORT của CloudWatch. | 02/09/2026 | 02/09/2026 | [Understanding Lambda Cold Starts](https://aws.amazon.com/blogs/compute/operating-lambda-performance-optimization-part-1/) | Quá trình nạp thư viện xgboost và scipy vào bộ nhớ là nguyên nhân chính gây trễ trong lần chạy đầu tiên. |
| Thứ 5 | Áp dụng kỹ thuật khởi tạo đối tượng mô hình tại phạm vi biến toàn cục bên ngoài hàm lambda_handler. Khi môi trường thực thi được tái sử dụng, hàm bỏ qua bước đọc tệp từ ổ đĩa. | 03/09/2026 | 03/09/2026 | [Reusing Execution Context in Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) | Thời gian suy luận ở trạng thái Warm Start giảm từ 1420 ms xuống mức 11 đến 14 ms. |
| Thứ 6 | Thực hiện đo lường hiệu năng theo các mức dung lượng RAM: Mức 256 MB ghi nhận thời gian Warm 35 ms và RAM sử dụng 148 MB; mức 512 MB ghi nhận thời gian Warm 12 ms và RAM sử dụng 152 MB; mức 1024 MB ghi nhận thời gian Warm 11 ms. | 04/09/2026 | 04/09/2026 | [AWS Lambda Memory and Computing Power](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-common.html) | Lựa chọn mức cấu hình 512 MB RAM và thời gian chờ 5 giây làm tiêu chuẩn triển khai. |
| Thứ 7 và Chủ nhật | Kiểm thử gửi liên tiếp 50 yêu cầu bằng mã nguồn Python để đo lường độ trễ phân vị P95 và P99. Cấu hình các biến môi trường phục vụ việc kết nối các dịch vụ phụ trợ. | 05/09/2026 | 06/09/2026 | [Configuring Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html) | Độ trễ phân vị P95 duy trì ổn định ở mức 13.2 ms. |

---

### 3. Thao tác kỹ thuật và mã lệnh cấu hình thực tế

#### 3.1. Lệnh tạo IAM Role và khởi tạo hàm Lambda
```bash
# 1. Tạo IAM Role với Trust Policy cho Lambda
aws iam create-role \
    --role-name PhishingDetectionLambdaRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": { "Service": "lambda.amazonaws.com" },
                "Action": "sts:AssumeRole"
            }
        ]
    }'

# 2. Gán quyền ghi log CloudWatch
aws iam attach-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# 3. Khởi tạo hàm Lambda từ Container Image ECR
aws lambda create-function \
    --function-name PhishingDetectionXGBoostContainer \
    --package-type Image \
    --code ImageUri=803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest \
    --role arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole \
    --memory-size 512 \
    --timeout 5 \
    --environment "Variables={CONFIDENCE_THRESHOLD=0.9,DYNAMODB_TABLE=PhishingDetectionLogs,SNS_TOPIC_ARN=arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic}" \
    --region ap-southeast-1
```

#### 3.2. Dữ liệu đo lường thực tế từ CloudWatch Report Log
```text
REPORT RequestId: 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d
Duration: 12.34 ms
Billed Duration: 13 ms
Memory Size: 512 MB
Max Memory Used: 154 MB
Init Duration: 1380.25 ms
```
Chỉ số Init Duration thể hiện thời gian khởi tạo môi trường ban đầu, chỉ phát sinh trong lần gọi đầu tiên khi microVM bắt đầu hoạt động.

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Lỗi Task timed out sau 3 giây trong lần gọi Cold Start đầu tiên.
  * Hiện tượng: Khi áp dụng mức thời gian chờ mặc định 3 giây của Lambda, yêu cầu đầu tiên bị hủy và API Gateway trả về mã lỗi 504 Gateway Timeout.
  * Nguyên nhân: Ở lần chạy đầu tiên, dịch vụ phải khởi động microVM Firecracker, tải ảnh từ ECR, nạp hệ điều hành Python và đọc các tệp mô hình vào bộ nhớ. Quá trình này mất từ 1.4 đến 1.8 giây, cộng với thời gian suy luận khiến tổng thời lượng vượt quá mốc 3 giây.
  * Giải pháp xử lý: Nâng mức thời gian chờ Timeout lên 5 giây bằng lệnh cập nhật cấu hình:
    ```bash
    aws lambda update-function-configuration \
        --function-name PhishingDetectionXGBoostContainer \
        --timeout 5 \
        --region ap-southeast-1
    ```
    Sau khi điều chỉnh lên 5 giây, tiến trình khởi động lần đầu diễn ra bình thường và các lần gọi tiếp theo phản hồi trong khoảng 12 ms.

---

### 5. Kết quả hoàn thành

1. Triển khai hàm Lambda PhishingDetectionXGBoostContainer hoạt động ổn định tại khu vực Singapore.
2. Đo lường thời gian suy luận Warm Start ghi nhận dưới 15 ms, mức tiêu thụ bộ nhớ thực tế đạt 154 MB trên tổng dung lượng cấp phát 512 MB.
3. Tách biệt mã nguồn và tham số thông qua cấu hình biến môi trường Lambda Environment Variables.
