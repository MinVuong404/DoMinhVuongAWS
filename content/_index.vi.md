---
title: "Báo cáo thực tập và Workshop kỹ thuật"
date: 2026-08-03
weight: 1
chapter: false
---

# Báo cáo thực tập tốt nghiệp và Workshop kỹ thuật

### Đề tài: Nghiên cứu dịch vụ Điện toán đám mây Amazon Web Services và Triển khai Kiến trúc Serverless
Dự án ứng dụng: Hệ thống nhận diện và cảnh báo thư điện tử lừa đảo, tên kỹ thuật Phishing Email Detection, kiến trúc Serverless trên AWS tích hợp mô hình học máy và tiện ích mở rộng Chrome Extension cho Gmail.

---

### 1. Thông tin sinh viên và cơ sở đào tạo
* Họ và tên sinh viên: Đỗ Minh Vương
* Mã số sinh viên: 0030268
* Lớp chuyên ngành: 68CNMHT
* Chuyên ngành đào tạo: Kỹ thuật Hệ thống và Mạng máy tính
* Khoa: Công nghệ Thông tin
* Cơ sở đào tạo: Trường Đại học Xây dựng Hà Nội
* Số điện thoại liên hệ: 0987.654.321
* Email liên hệ: vuong.dmforwork@gmail.com
* Giảng viên hướng dẫn: ThS. Nguyễn Việt Nhật, Khoa Công nghệ Thông tin, Trường Đại học Xây dựng Hà Nội

### 2. Đơn vị tiếp nhận thực tập
* Đơn vị tiếp nhận: Công ty TNHH Amazon Web Services Việt Nam, AWS Việt Nam
* Địa chỉ pháp lý: Tầng 36, Tòa nhà Bitexco Financial Tower, Số 2 Hải Triều, Phường Bến Nghé, Quận 1, TP. Hồ Chí Minh
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, Số 36 Cát Linh, Phường Ô Chợ Dừa, Quận Đống Đa, TP. Hà Nội
* Vị trí thực tập: Cloud Solutions Architect và Systems Engineering Intern
* Chương trình đào tạo: First Cloud AI Journey, mã chương trình FCAJ Workforce Bootcamp 2026
* Cán bộ phụ trách: Nguyễn Gia Hưng, Senior Solutions Architect, AWS Việt Nam
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, Solutions Architect, AWS Việt Nam
* Thời gian thực tập: 03/08/2026 đến 27/09/2026 gồm 8 tuần thực tập tập trung và 4 tuần mở rộng từ tuần 9 đến tuần 12.

---

{{% notice info %}}
Tóm tắt giải pháp kỹ thuật dự án Phishing Email Detection on AWS:  
* Bối cảnh: Tấn công lừa đảo qua email gây thiệt hại tài chính cho doanh nghiệp. Giải pháp kiểm tra truyền thống dùng máy chủ EC2 chạy liên tục gây phát sinh chi phí duy trì và có độ trễ cảnh báo cao.
* Kiến trúc giải pháp: Hệ thống phân loại văn bản thời gian thực sử dụng mô hình XGBoost kết hợp TruncatedSVD với độ chính xác đạt trên 97%, đóng gói dạng Docker Container lưu trữ tại Amazon ECR, thực thi trên AWS Lambda, tiếp nhận yêu cầu qua Amazon API Gateway v2 HTTP API có cấu hình CORS.
* Nhật ký và cảnh báo: Dữ liệu kiểm toán ghi nhận thời gian thực vào CSDL NoSQL Amazon DynamoDB. Khi phát hiện email độc hại với xác suất từ 90% trở lên, hệ thống kích hoạt Amazon SNS gửi email cảnh báo đến quản trị viên. Toàn bộ hoạt động được giám sát qua Amazon CloudWatch.
* Thông số vận hành: Độ trễ suy luận Warm Start dưới 15 ms, chi phí hạ tầng 0.00 USD trong hạn mức AWS Free Tier, tự động co giãn theo số lượng yêu cầu đồng thời.
{{% /notice %}}

---

### Sơ đồ kiến trúc tổng quan hệ thống

![Sơ đồ Kiến trúc Tổng quan Hệ thống](/images/architecture/serverless-system-architecture.svg)

---

### Mục lục báo cáo và tài liệu thực hành

1. [Nhật ký công việc 12 tuần](1-worklog/)  
   Chi tiết 8 tuần thực tập tại Văn phòng AWS Hà Nội và kế hoạch công tác mở rộng từ tuần 9 đến tuần 12.
2. [Đề xuất dự án kỹ thuật](2-proposal/)  
   Bối cảnh bài toán, mục tiêu kỹ thuật, kiến trúc Serverless, dự toán chi phí và phương án quản trị rủi ro.
3. [Sự kiện công nghệ tham gia](3-eventparticipated/)  
   Báo cáo thu hoạch sự kiện AWS Vietnam Community Meetup với chủ đề AI Revolution và Open Claw tại Văn phòng AWS Hà Nội.
4. [Tài liệu thực hành Workshop](4-workshop/)  
   Hướng dẫn 8 bài thực hành triển khai hệ thống gồm Docker, ECR, Lambda, API Gateway, DynamoDB, SNS, Chrome Extension và CloudWatch.
5. [Tự đánh giá kết quả thực tập](5-self-evaluation/)  
   Đánh giá kết quả thực tập theo các tiêu chí của cơ sở đào tạo.
6. [Đóng góp ý kiến](6-feedback/)  
   Đánh giá trải nghiệm thực tập và kiến nghị đối với chương trình đào tạo.