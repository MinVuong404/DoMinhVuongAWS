---
title: "Nhật ký Tuần 2"
date: 2026-08-10
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Nhật ký Tuần 2: Khảo sát dịch vụ cốt lõi và nghiên cứu kiến trúc Serverless

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 10/08/2026 đến 16/08/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Khảo sát 8 dịch vụ AWS gồm Amazon EC2, Amazon S3, AWS IAM, AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon SNS và Amazon CloudWatch.
  2. Xây dựng bảng so sánh giữa mô hình máy chủ ảo EC2 với cơ sở dữ liệu quan hệ và mô hình Serverless sử dụng Lambda kết hợp DynamoDB.
  3. Phân tích tổng chi phí sở hữu và chính sách AWS Free Tier nhằm xác định phương án chi phí 0 USD cho hệ thống ở quy mô thử nghiệm.
  4. Xác định các giới hạn tài nguyên của AWS Lambda gồm kích thước gói tải lên tối đa 6 MB, dung lượng lưu trữ tạm thời từ 512 MB đến 10 GB, giới hạn giải nén tệp Zip là 250 MB và giới hạn Container Image trên ECR là 10 GB.
  5. Cài đặt môi trường Python 3.12 trên máy tính cá nhân và thử nghiệm các bước tiền xử lý văn bản.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Nghiên cứu dịch vụ máy chủ ảo Amazon EC2, hệ thống ảo hóa Nitro System và các dòng máy chủ. Tạo S3 Bucket, cấu hình mã hóa SSE-S3 và bật tính năng chặn truy cập công khai Block Public Access. | 10/08/2026 | 10/08/2026 | [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)<br>[Amazon S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html) | Khởi tạo S3 Bucket lưu trữ tài liệu kỹ thuật của dự án. |
| Thứ 3 | Tìm hiểu vòng đời thực thi của AWS Lambda gồm hệ thống ảo hóa Firecracker MicroVM, môi trường thực thi Execution Environment và sự khác biệt giữa Cold Start và Warm Start. Viết hàm Lambda Python in dữ liệu sự kiện và cấu trúc ngữ cảnh. | 11/08/2026 | 11/08/2026 | [AWS Lambda Execution Environment](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-context.html)<br>[Firecracker MicroVM Whitepaper](https://firecracker-microvm.github.io/) | Cơ chế giữ ấm biến toàn cục trong bộ nhớ giúp giảm thời gian phản hồi từ 1200 ms xuống dưới 15 ms ở các lần gọi tiếp theo. |
| Thứ 4 | Thử nghiệm triển khai mô hình học máy lên Lambda bằng tệp Zip chứa thư viện scikit-learn, scipy và xgboost. Gặp lỗi vượt ngưỡng dung lượng khi thư viện sau giải nén có kích thước 380 MB, vượt quá giới hạn 250 MB của gói triển khai Zip. | 12/08/2026 | 12/08/2026 | [Lambda Deployment Packages](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html)<br>[AWS Lambda Container Image Support](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/) | Chuyển sang giải pháp đóng gói Docker Container Image hỗ trợ dung lượng tối đa 10 GB thông qua Amazon ECR. |
| Thứ 5 | Lập bảng tính toán chi phí: Máy chủ EC2 t3.medium chạy liên tục phát sinh chi phí khoảng 30 USD mỗi tháng, trong khi Lambda kết hợp DynamoDB miễn phí với quy mô dưới 1.000.000 yêu cầu mỗi tháng. Báo cáo phương án kiến trúc với cán bộ hướng dẫn. | 13/08/2026 | 13/08/2026 | [AWS Pricing Calculator](https://calculator.aws/)<br>[AWS Free Tier Terms](https://aws.amazon.com/free/) | Cán bộ hướng dẫn thống nhất lựa chọn kiến trúc Serverless để tối ưu chi phí hạ tầng. |
| Thứ 6 | Chuẩn bị tập dữ liệu email lừa đảo từ nguồn mở SpamAssassin và Phishing Corpus. Huấn luyện thử nghiệm mô hình phân loại nhị phân sử dụng TF-IDF, TruncatedSVD 300 chiều và thuật toán XGBoost trên máy trạm. | 14/08/2026 | 14/08/2026 | [Scikit-learn Documentation](https://scikit-learn.org/)<br>[XGBoost Python API](https://xgboost.readthedocs.io/) | Mô hình đạt độ chính xác Accuracy 97.4%, chỉ số F1-score đạt 0.96. Tổng kích thước 3 tệp trọng số là 38 MB. |
| Thứ 7 và Chủ nhật | Nghiên cứu cách viết Dockerfile cho Lambda Python sử dụng Base Image public.ecr.aws/lambda/python:3.12. Viết mã tiền xử lý làm sạch văn bản bằng biểu thức chính quy và chuẩn hóa đường dẫn web. | 15/08/2026 | 16/08/2026 | [AWS Lambda Base Images for Docker](https://gallery.ecr.aws/lambda/python)<br>[Python Regex Module Docs](https://docs.python.org/3/library/re.html) | Hoàn thành tệp mã nguồn tiền xử lý và quy trình biến đổi đặc trưng văn bản. |

---

### 3. So sánh kỹ thuật giữa các mô hình kiến trúc

| Tiêu chí kỹ thuật | Mô hình máy chủ ảo EC2 và RDS | Mô hình Serverless Lambda và DynamoDB | Lợi thế trong dự án |
| :--- | :--- | :--- | :--- |
| Chi phí duy trì khi không có tải | Trả phí thuê máy chủ liên tục từ 35 đến 50 USD mỗi tháng kể cả khi không có yêu cầu. | Chi phí 0.00 USD mỗi tháng trong hạn mức Free Tier do chỉ tính phí khi có yêu cầu xử lý. | Không phát sinh chi phí vận hành hạ tầng thử nghiệm. |
| Cơ chế co giãn | Cần cấu hình Auto Scaling Group, mất từ 2 đến 5 phút để khởi chạy phiên bản máy chủ mới. | Tự động mở rộng tức thời theo số lượng yêu cầu đồng thời. | Đáp ứng lưu lượng thay đổi khi nhiều người dùng gửi yêu cầu cùng lúc. |
| Quản trị hạ tầng | Tự cập nhật bản vá hệ điều hành, cấu hình tường lửa và cổng kết nối. | AWS quản lý hạ tầng phần cứng và môi trường thực thi microVM. | Giảm thiểu thời gian vận hành hệ thống. |
| Kích thước gói triển khai | Phụ thuộc vào dung lượng ổ đĩa EBS gán kèm. | Hỗ trợ tối đa 10 GB khi sử dụng Docker Container Image qua Amazon ECR. | Đáp ứng dung lượng của các thư viện scikit-learn và xgboost. |

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Lỗi InvalidParameterValueException do kích thước tệp giải nén vượt quá 262144000 bytes khi tải tệp Zip lên Lambda.
  * Hiện tượng: Gói nén thư viện site-packages gồm numpy, scipy, scikit-learn và xgboost có dung lượng tệp nén 82 MB, nhưng khi giải nén đạt 312 MB, khiến bảng điều khiển Lambda từ chối tiếp nhận vì vượt mức trần 250 MB.
  * Nguyên nhân: Lambda áp dụng giới hạn cứng 250 MB cho tệp mã nguồn triển khai qua hình thức nén Zip trực tiếp. Các thư viện tính toán khoa học chứa mã nhị phân biên dịch sẵn chiếm dung lượng lớn.
  * Giải pháp xử lý: Chuyển sang đóng gói theo chuẩn OCI Container Image bằng Docker. Lambda hỗ trợ nạp Container Image với dung lượng lên đến 10 GB từ kho lưu trữ Amazon ECR.

---

### 5. Kết quả hoàn thành

1. Hoàn thành báo cáo so sánh kiến trúc giữa máy chủ truyền thống và giải pháp Serverless.
2. Huấn luyện mô hình XGBoost đạt độ chính xác Accuracy 97.4%, xuất 3 tệp trọng số model.joblib, tfidf.joblib và svd.joblib với tổng kích thước 38 MB.
3. Hoàn thiện mã nguồn trích xuất dữ liệu, làm sạch mã HTML và chuẩn hóa chuỗi ký tự đầu vào.
