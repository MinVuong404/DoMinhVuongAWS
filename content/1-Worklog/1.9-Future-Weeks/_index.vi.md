---
title: "Kế hoạch Tuần 9 - 12"
date: 2026-09-28
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

# Lộ trình nghiên cứu và mở rộng hệ thống từ tuần 9 đến tuần 12

Lộ trình công tác 4 tuần mở rộng từ tuần 9 đến tuần 12 nhằm định hướng nâng cấp hệ thống phát hiện email lừa đảo lên quy mô doanh nghiệp với các mục tiêu kỹ thuật cụ thể:

---

### Kế hoạch tổng quan 4 tuần mở rộng

| Tuần | Thời gian | Định hướng kỹ thuật | Dịch vụ AWS sử dụng | Kết quả dự kiến |
| :---: | :---: | :--- | :--- | :--- |
| Tuần 9 | 28/09 đến 04/10/2026 | Bảo vệ vành đai và giới hạn tần suất yêu cầu | AWS WAF, API Gateway v2 | Giới hạn tần suất 20 yêu cầu trong 5 phút từ một địa chỉ IP, ngăn chặn tấn công từ chối dịch vụ. |
| Tuần 10 | 05/10 đến 11/10/2026 | Lưu trữ và phân tích dữ liệu tập trung | Amazon S3, DynamoDB Streams, AWS Glue, Athena | Lưu trữ dữ liệu dạng Parquet trên S3, truy vấn dữ liệu bằng SQL qua Athena. |
| Tuần 11 | 12/10 đến 18/10/2026 | Tự động hóa quy trình huấn luyện lại mô hình | AWS Step Functions, Amazon SageMaker, EventBridge | Đường ống tự động huấn luyện lại và cập nhật mô hình khi độ chính xác suy giảm. |
| Tuần 12 | 19/10 đến 25/10/2026 | Rà soát kiến trúc theo chuẩn Well-Architected | AWS Well-Architected Tool, CloudWatch | Báo cáo rà soát 6 trụ cột kiến trúc, hoàn thiện hồ sơ nghiệm thu. |

---

### Chi tiết kế hoạch công tác từng tuần

#### 1. Tuần 9, từ 28/09/2026 đến 04/10/2026: Tăng cường bảo mật với AWS WAF và Rate Limiting
* Bối cảnh: Điểm cuối API Gateway mở công khai có rủi ro bị gửi yêu cầu liên tục, gây tiêu hao tài nguyên xử lý và phát sinh chi phí Lambda ngoài dự toán.
* Mục tiêu kỹ thuật:
  * Triển khai dịch vụ tường lửa AWS WAF đặt trước Amazon API Gateway v2.
  * Cấu hình quy tắc giới hạn tần suất Rate-based Rule ở mức tối đa 20 yêu cầu trong 5 phút trên một địa chỉ IP nguồn.
  * Kích hoạt nhóm quy tắc quản lý AWS Managed Rules gồm các bộ lọc chống SQL Injection, Cross-Site Scripting và danh sách địa chỉ IP có độ uy tín thấp.
* Kết quả dự kiến: Hệ thống lọc và ngăn chặn các yêu cầu gửi tự động từ mạng máy tính ma.

#### 2. Tuần 10, từ 05/10/2026 đến 11/10/2026: Xây dựng kho lưu trữ dữ liệu email trên Amazon S3 và AWS Glue
* Bối cảnh: Bảng DynamoDB duy trì dữ liệu trong thời hạn ngắn để tối ưu chi phí NoSQL. Hệ thống cần một kho lưu trữ dài hạn để lưu vết các mẫu email phục vụ phân tích nghiệp vụ.
* Mục tiêu kỹ thuật:
  * Bật tính năng DynamoDB Streams để thu thập các bản ghi mới phát sinh.
  * Sử dụng Amazon Kinesis Data Firehose để chuyển tiếp dòng dữ liệu vào kho lưu trữ đối tượng Amazon S3.
  * Chuyển đổi định dạng dữ liệu sang Apache Parquet kết hợp nén Snappy để giảm 80% dung lượng lưu trữ trên đĩa.
  * Cấu hình danh mục dữ liệu AWS Glue Data Catalog và sử dụng Amazon Athena để truy vấn dữ liệu bằng ngôn ngữ SQL mà không cần khởi tạo máy chủ cơ sở dữ liệu.
* Kết quả dự kiến: Thiết lập hạ tầng lưu trữ dữ liệu tập trung với chi phí thấp trên Amazon S3.

#### 3. Tuần 11, từ 12/10/2026 đến 18/10/2026: Tự động hóa quy trình học máy với Amazon SageMaker
* Bối cảnh: Các mẫu thư lừa đảo mới xuất hiện liên tục theo thời gian làm giảm độ chính xác của mô hình ban đầu.
* Mục tiêu kỹ thuật:
  * Xây dựng máy trạng thái AWS Step Functions kích hoạt tự động theo chu kỳ tháng hoặc khi tập dữ liệu mẫu mới trong S3 đạt mức 50.000 bản ghi.
  * Khởi chạy tác vụ huấn luyện Amazon SageMaker Training Job trên máy ảo Spot Instances để giảm 70% chi phí tính toán khi huấn luyện lại mô hình XGBoost.
  * Đánh giá mô hình mới: Nếu chỉ số F1-Score cao hơn mô hình đang chạy, dịch vụ AWS CodeBuild tự động biên dịch lại Docker Image và tải lên Amazon ECR.
  * Cập nhật mã nguồn hàm Lambda sang mã băm Image Digest mới bằng lệnh aws lambda update-function-code.
* Kết quả dự kiến: Tự động hóa chu trình huấn luyện liên tục và triển khai liên tục cho mô hình học máy.

#### 4. Tuần 12, từ 19/10/2026 đến 25/10/2026: Đánh giá kiến trúc AWS Well-Architected và nghiệm thu
* Bối cảnh: Rà soát toàn bộ hệ thống theo các tiêu chuẩn thiết kế kiến trúc đám mây trước khi hoàn thiện hồ sơ nghiệm thu.
* Mục tiêu kỹ thuật:
  * Thực hiện đánh giá kiến trúc AWS Well-Architected Review theo 6 trụ cột gồm Vận hành xuất sắc, An toàn bảo mật, Độ tin cậy, Hiệu năng, Tối ưu chi phí và Tính bền vững.
  * Lập danh sách các vấn đề rủi ro mức cao và mức trung bình để điều chỉnh cấu hình hệ thống.
  * Hoàn thiện hồ sơ báo cáo thực tập, tổng hợp nhận xét của đơn vị tiếp nhận và chuẩn bị bảo vệ đồ án tốt nghiệp.
* Kết quả dự kiến: Báo cáo nghiệm thu hoàn chỉnh, đáp ứng các tiêu chuẩn kỹ thuật đề ra.
