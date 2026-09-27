---
title: "Workshop"
date: 2026-08-25
weight: 4
chapter: false
pre: " <b> 4. </b> "
aliases:
  - /5-workshop/
---

# Hướng dẫn thực hành triển khai hệ thống phát hiện email lừa đảo kiến trúc Serverless trên AWS

#### Tổng quan nội dung thực hành
Chuỗi bài thực hành hướng dẫn các bước triển khai hệ thống từ thiết kế kiến trúc, đóng gói mô hình học máy vào Docker Container, triển khai lên AWS Lambda thông qua Amazon ECR, tiếp nhận yêu cầu qua Amazon API Gateway v2, lưu trữ nhật ký kiểm toán vào Amazon DynamoDB, phát cảnh báo qua Amazon SNS, tích hợp tiện ích mở rộng Chrome Extension cho Gmail và giám sát hệ thống bằng Amazon CloudWatch.

{{% notice info %}}
* Tên dự án: AWS Serverless Real-time Phishing Detection and Alert System
* Kho mã nguồn: [https://github.com/dov005/AWS-Serverless-Phishing-Detection](https://github.com/dov005/AWS-Serverless-Phishing-Detection)
* Kiến trúc: Serverless gồm Amazon ECR, AWS Lambda, Amazon API Gateway v2, Amazon DynamoDB, Amazon SNS, Amazon CloudWatch và AWS IAM kết hợp tiện ích mở rộng Chrome Extension Manifest V3.
* Khu vực triển khai: Khu vực Singapore ap-southeast-1 nhằm duy trì độ trễ mạng dưới 35 ms tới người dùng tại Việt Nam.
* Chi phí vận hành: 0.00 USD mỗi tháng trong phạm vi hạn mức AWS Free Tier.
{{% /notice %}}

---

### Sơ đồ kiến trúc tổng thể hệ thống

![Sơ đồ Kiến trúc Tổng thể Hệ thống](/images/architecture/serverless-system-architecture.svg)

---

#### Danh sách các bài thực hành

1. [4.1. Tổng quan kiến trúc và chuẩn bị môi trường](4.1-architecture-environment/)
   * 4.1.1. Phân tích luồng dữ liệu và thông số kỹ thuật mô hình học máy XGBoost và TruncatedSVD
   * 4.1.2. Khởi tạo và kiểm tra công cụ phát triển gồm AWS CLI v2, Docker Desktop và Python 3.12
   * 4.1.3. Thiết lập IAM Execution Role PhishingDetectionLambdaRole theo chuẩn đặc quyền tối thiểu
2. [4.2. Đóng gói Container Docker và lưu trữ trên Amazon ECR](4.2-docker-ecr/)
   * 4.2.1. Cấu trúc mã nguồn và soạn thảo Dockerfile trên nền tảng Amazon Linux 2023
   * 4.2.2. Khởi tạo kho lưu trữ Amazon ECR riêng tư phishing-xgboost
   * 4.2.3. Xác thực Docker CLI, biên dịch ảnh và đẩy Container Image lên Amazon ECR
3. [4.3. Triển khai hàm AWS Lambda từ Container Image](4.3-lambda-deployment/)
   * 4.3.1. Khởi tạo hàm Lambda PhishingDetectionXGBoostContainer từ ECR Image
   * 4.3.2. Cấu hình tài nguyên 512 MB RAM, thời gian chờ 5 giây và thiết lập biến môi trường
   * 4.3.3. Tối ưu thời gian khởi động Cold Start bằng kỹ thuật nạp mô hình vào bộ nhớ đệm toàn cục
4. [4.4. Cấu hình cổng kết nối Amazon API Gateway v2](4.4-api-gateway-v2/)
   * 4.4.1. Khởi tạo HTTP API Gateway v2 và định tuyến POST /predict
   * 4.4.2. Thiết lập Lambda Proxy Integration và cấp quyền thực thi lambda:InvokeFunction
   * 4.4.3. Cấu hình cơ chế chia sẻ tài nguyên CORS cho trình duyệt web
5. [4.5. Tích hợp lưu trữ Amazon DynamoDB và cảnh báo Amazon SNS](4.5-dynamodb-sns/)
   * 4.5.1. Khởi tạo bảng NoSQL PhishingDetectionLogs theo hạn mức AWS Free Tier
   * 4.5.2. Khởi tạo SNS Topic PhishingAlertTopic và đăng ký nhận email cảnh báo sự cố
   * 4.5.3. Gán quyền truy cập dynamodb:PutItem và sns:Publish cho vai trò của Lambda
6. [4.6. Cài đặt Chrome Extension và kiểm thử trên Gmail](4.6-chrome-extension-testing/)
   * 4.6.1. Cấu trúc tiện ích Manifest V3 và cơ chế bóc tách dữ liệu DOM Gmail bằng MutationObserver
   * 4.6.2. Cài đặt tiện ích vào trình duyệt Chrome ở chế độ dành cho nhà phát triển
   * 4.6.3. Kiểm thử 20 kịch bản email: Xác minh độ chính xác và kiểm tra cảnh báo qua SNS
7. [4.7. Giám sát vận hành và cảnh báo sự cố với Amazon CloudWatch](4.7-cloudwatch-monitoring/)
   * 4.7.1. Phân tích luồng nhật ký và truy vấn hiệu năng trên CloudWatch Logs Insights
   * 4.7.2. Thiết lập bảng điều khiển tập trung PhishingDetection-Operations-Dashboard
   * 4.7.3. Cấu hình quy tắc cảnh báo CloudWatch Metric Alarms giám sát lỗi và độ trễ
8. [4.8. Dọn dẹp tài nguyên](4.8-resource-cleanup/)
   * 4.8.1. Hướng dẫn xóa tài nguyên thông qua AWS Management Console
   * 4.8.2. Kịch bản dọn dẹp tài nguyên tự động bằng tập lệnh dòng lệnh AWS CLI
