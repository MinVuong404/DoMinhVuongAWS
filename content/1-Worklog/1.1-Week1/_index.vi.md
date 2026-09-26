---
title: "Nhật ký Tuần 1"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Nhật ký Tuần 1: Khởi tạo môi trường, quản trị định danh IAM và nền tảng AWS Cloud

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 03/08/2026 đến 09/08/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Thiết lập tài khoản AWS thực hành theo trụ cột an toàn bảo mật của AWS Well-Architected Framework: Bật xác thực đa yếu tố MFA cho tài khoản Root, hủy các khóa truy cập Root Access Keys nếu tồn tại.
  2. Xây dựng mô hình phân quyền quản trị định danh AWS Identity and Access Management theo nguyên tắc đặc quyền tối thiểu.
  3. Cài đặt và cấu hình công cụ dòng lệnh AWS CLI v2 trên môi trường máy trạm, thiết lập khu vực mặc định ap-southeast-1 tại Singapore.
  4. Cấu hình kiểm soát chi phí tự động thông qua dịch vụ AWS Budgets và CloudWatch Billing Alarm kết hợp Amazon SNS gửi email cảnh báo khi chi phí chạm ngưỡng 5 USD.
  5. Nghiên cứu mô hình trách nhiệm chia sẻ và các tiêu chuẩn trong AWS Well-Architected Framework.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Tiếp nhận kế hoạch thực tập tại Văn phòng AWS Hà Nội. Khởi tạo tài khoản AWS thực hành trong chương trình First Cloud AI Journey. Kích hoạt tính năng xác thực đa yếu tố Virtual MFA cho tài khoản Root bằng ứng dụng Google Authenticator. Xác nhận không phát hành khóa Root Access Keys. | 03/08/2026 | 03/08/2026 | [Cloud Journey Training Portal](https://cloudjourney.awsstudygroup.com/)<br>[AWS Account Setup & Root MFA Best Practices](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-root-user.html) | Tài khoản kích hoạt hoàn tất, cấu hình bảo mật tài khoản gốc tuân thủ theo tiêu chuẩn CIS AWS Foundations Benchmark. |
| Thứ 3 | Cài đặt AWS CLI v2 trên máy tính cá nhân. Tạo người dùng IAM dev-vuong, phân quyền thông qua nhóm CloudArchitects. Cấu hình hồ sơ xác thực cục bộ trong thư mục aws với hai tệp credentials và config. | 04/08/2026 | 04/08/2026 | [AWS CLI v2 Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)<br>[IAM Best Practices & Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) | Lệnh aws sts get-caller-identity phản hồi thông tin định danh IAM thành công, khu vực trỏ về ap-southeast-1. |
| Thứ 4 | Kích hoạt tùy chọn nhận cảnh báo thanh toán trong mục Billing Preferences. Thiết lập hạn mức ngân sách AWS Budget ở mức 10 USD mỗi tháng cho tài khoản thực hành. Tạo CloudWatch Metric Alarm theo dõi chỉ số EstimatedCharges với ngưỡng cảnh báo 5 USD tại khu vực us-east-1. | 05/08/2026 | 05/08/2026 | [AWS Budgets Documentation](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)<br>[CloudWatch Billing Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) | Cấu hình chủ đề Amazon SNS Billing-Alerts-Topic, xác nhận đăng ký nhận thư cảnh báo qua địa chỉ vuong.dmforwork@gmail.com. |
| Thứ 5 | Nghiên cứu hạ tầng toàn cầu của AWS gồm Region, Availability Zones và Edge Locations. Đo lường chỉ số độ trễ truyền dữ liệu khứ hồi từ mạng Việt Nam tới các khu vực ap-southeast-1 và ap-east-1. Đọc tài liệu thiết kế hệ thống theo hai trụ cột Security và Cost Optimization. | 06/08/2026 | 06/08/2026 | [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) | Lựa chọn khu vực Singapore ap-southeast-1 với độ trễ ghi nhận trung bình là 32 ms. |
| Thứ 6 | Tham gia phiên họp định hướng kỹ thuật cùng cán bộ hướng dẫn và kiến trúc sư giải pháp. Trình bày đề xuất đề tài phát hiện và cảnh báo email lừa đảo kiến trúc Serverless trên AWS. Tiếp nhận góp ý chuyên môn về phương án đóng gói mô hình học máy vào container để chạy trên AWS Lambda. | 07/08/2026 | 07/08/2026 | [Serverless Machine Learning on AWS](https://aws.amazon.com/blogs/compute/)<br>[Working with Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) | Đề tài được thông qua, thống nhất sử dụng kiến trúc gồm Docker Container, ECR, AWS Lambda, API Gateway v2, DynamoDB và SNS. |
| Thứ 7 và Chủ nhật | Hoàn thành các bài kiểm tra trắc nghiệm trên cổng đào tạo Cloud Journey. Nghiên cứu cơ chế đánh giá chính sách truy cập IAM Policies, Resource-based Policies và IAM Roles. | 08/08/2026 | 09/08/2026 | [AWS Skill Builder Training](https://explore.skillbuilder.aws/)<br>[IAM Policies Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) | Hoàn thành bài kiểm tra đánh giá kiến thức cơ bản đầu khóa. |

---

### 3. Thao tác kỹ thuật và mã lệnh cấu hình thực tế

#### 3.1. Cấu hình công cụ dòng lệnh AWS CLI v2
```bash
# Cấu hình Region mặc định và định dạng đầu ra
aws configure set default.region ap-southeast-1
aws configure set default.output json

# Xác thực định danh hiện tại và kiểm tra quyền hạn thực thi
aws sts get-caller-identity
```
Dữ liệu JSON trả về:
```json
{
    "UserId": "AIDA4TESTVUONGFCAJ01",
    "Account": "803146828520",
    "Arn": "arn:aws:iam::803146828520:user/dev-vuong"
}
```

#### 3.2. Khởi tạo tài khoản IAM User và Group theo đặc quyền tối thiểu
```bash
# 1. Tạo nhóm phát triển đám mây
aws iam create-group --group-name CloudArchitects

# 2. Gán chính sách quyền hạn cho công việc phát triển, không sử dụng quyền AdministratorAccess
aws iam attach-group-policy --group-name CloudArchitects \
    --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# 3. Tạo tài khoản người dùng và thêm vào nhóm
aws iam create-user --user-name dev-vuong
aws iam add-user-to-group --user-name dev-vuong --group-name CloudArchitects
```

#### 3.3. Thiết lập cảnh báo chi phí tự động qua CloudWatch và SNS
```bash
# Tạo SNS Topic nhận thông báo chi phí tại khu vực us-east-1 theo quy định của Billing
aws sns create-topic --name Billing-Alerts-Topic --region us-east-1

# Đăng ký nhận thông báo qua Email
aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:803146828520:Billing-Alerts-Topic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region us-east-1

# Tạo CloudWatch Alarm cảnh báo khi chi phí ước tính vượt quá 5.0 USD
aws cloudwatch put-metric-alarm \
    --alarm-name "Billing-Threshold-Over-5USD" \
    --metric-name EstimatedCharges \
    --namespace AWS/Billing \
    --statistic Maximum \
    --period 21600 \
    --threshold 5.0 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=Currency,Value=USD \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:803146828520:Billing-Alerts-Topic \
    --region us-east-1
```

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố 1: Lỗi SignatureDoesNotMatch khi thực thi lệnh trên AWS CLI máy cục bộ.
  * Hiện tượng: Khi chạy lệnh aws ec2 describe-regions hoặc aws sts get-caller-identity, màn hình hiển thị lỗi chữ ký hết hạn, Signature expired, do chênh lệch thời gian giữa máy trạm và máy chủ xác thực.
  * Nguyên nhân: Cơ chế xác thực AWS Signature Version 4 từ chối các yêu cầu có mốc thời gian lệch quá 15 phút so với máy chủ thời gian AWS NTP Server nhằm ngăn chặn tấn công gửi lại. Đồng hồ hệ thống trên máy tính trôi lệch hơn 5 phút so với thời gian mạng chuẩn.
  * Giải pháp xử lý: Chạy lệnh đồng bộ lại thời gian Windows Time Service bằng quyền quản trị:
    ```powershell
    net start w32time
    w32tm /resync /force
    ```
    Sau khi đồng bộ thời gian, các lệnh CLI hoạt động bình thường.

* Sự cố 2: Chỉ số EstimatedCharges không xuất hiện trong CloudWatch.
  * Hiện tượng: Lệnh tạo cảnh báo báo lỗi không tìm thấy chỉ số EstimatedCharges trong không gian tên AWS/Billing.
  * Nguyên nhân: Mặc định tài khoản AWS chưa bật tính năng thu thập dữ liệu hóa đơn vào CloudWatch. Ngoài ra, dữ liệu thanh toán toàn cầu của AWS chỉ được lưu trữ tại khu vực us-east-1, không hỗ trợ tại khu vực ap-southeast-1.
  * Giải pháp xử lý: Đăng nhập tài khoản Root, truy cập Billing Preferences, kích hoạt tùy chọn Receive CloudWatch Billing Alerts, sau đó thêm tham số chỉ định khu vực us-east-1 khi khởi tạo cảnh báo. Sau 15 phút, chỉ số hiển thị đầy đủ và trạng thái cảnh báo chuyển về mức bình thường.

---

### 5. Kết quả hoàn thành

1. Tài khoản AWS thực hành được thiết lập bảo mật theo chuẩn CIS Benchmark với xác thực đa yếu tố cho tài khoản gốc và phân quyền thông qua IAM User dev-vuong.
2. Thiết lập hệ thống giám sát ngân sách tự động nhằm kiểm soát mức chi phí phát sinh trong thời gian thực tập.
3. Cài đặt các công cụ phát triển gồm AWS CLI v2, Python 3.12 và Docker Desktop trên máy trạm cá nhân.
4. Đề xuất đề tài ứng dụng phát hiện email lừa đảo kiến trúc Serverless được cán bộ hướng dẫn thông qua.
