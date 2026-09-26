---
title: "Đề xuất dự án"
date: 2026-08-07
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Bản đề xuất dự án kỹ thuật

### Tên dự án: Hệ thống nhận diện và cảnh báo thư điện tử lừa đảo kiến trúc Serverless trên AWS

---

### 1. Tổng quan dự án
Thư điện tử là kênh trao đổi thông tin chính trong hoạt động doanh nghiệp, đồng thời là phương thức tấn công phổ biến của tội phạm mạng. Các hình thức lừa đảo qua thư điện tử như giả mạo tổ chức tài chính, giả mạo lãnh đạo doanh nghiệp hoặc thông báo khóa tài khoản gây thiệt hại tài chính và rò rỉ dữ liệu.

Dự án triển khai giải pháp kỹ thuật kết hợp mô hình học máy XGBoost, kỹ thuật giảm chiều TruncatedSVD và kiến trúc Serverless trên nền tảng AWS. Hệ thống cung cấp khả năng phân loại thư điện tử thông qua tiện ích mở rộng Chrome Extension tích hợp trực tiếp trên giao diện web Gmail.

---

### 2. Vấn đề cần giải quyết

{{% notice warning %}}
Thực trạng an ninh thông tin: Các cuộc tấn công giả mạo sử dụng tên miền tương tự, nội dung tạo tâm lý khẩn cấp và yêu cầu mã OTP hoặc mật khẩu. Người dùng cuối có nguy cơ truy cập liên kết độc hại do khó phân biệt bằng mắt thường trên giao diện hộp thư.
{{% /notice %}}

#### Bảng so sánh giải pháp truyền thống và kiến trúc Serverless:

| Tiêu chí đánh giá | Giải pháp Email Gateway truyền thống | Kiến trúc Serverless trong dự án |
| :--- | :--- | :--- |
| Chi phí hạ tầng và vận hành | Duy trì máy chủ EC2 liên tục với chi phí ước tính 30 đến 50 USD mỗi tháng kể cả khi không có tải. | Chi phí 0.00 USD trong hạn mức AWS Free Tier, tự co giãn về 0 khi không có lưu lượng truy cập. |
| Thời điểm cảnh báo | Xử lý theo lô tại Mail Transfer Agent, độ trễ nhận diện cao. | Xử lý thời gian thực với độ trễ dưới 200 ms ngay khi người dùng mở thư trên Gmail. |
| Vận hành và mở rộng | Phải cấu hình hệ điều hành, cập nhật bản vá bảo mật định kỳ, giới hạn khả năng mở rộng tức thời. | Không cần quản trị máy chủ, hệ thống tự động mở rộng theo số lượng yêu cầu đồng thời. |
| Kiểm toán và cảnh báo sự cố | Ghi nhật ký phân tán trên máy chủ, thiếu cơ chế thông báo tức thời đến đội ngũ vận hành. | Amazon DynamoDB lưu vết toàn bộ yêu cầu, Amazon SNS gửi cảnh báo email khi xác suất lừa đảo đạt từ 90% trở lên. |

---

### 3. Mục tiêu dự án

1. Hiệu năng mô hình phân loại: Xây dựng mô hình phân loại nhị phân đạt độ chính xác Accuracy từ 97% trở lên, chỉ số F1-Score đạt từ 0.95 trở lên trên tập dữ liệu tiếng Việt và tiếng Anh.
2. Triển khai kiến trúc Serverless: Vận hành toàn bộ dịch vụ tính toán và lưu trữ không cần quản lý máy chủ vật lý hay máy chủ ảo, tự động mở rộng theo lưu lượng thực tế.
3. Độ trễ xử lý: Thời gian xử lý của hàm Lambda ở trạng thái Warm Start dưới 15 ms, tổng thời gian phản hồi toàn trình đến giao diện người dùng dưới 200 ms.
4. Tối ưu chi phí vận hành: Vận hành toàn bộ hệ thống trong hạn mức AWS Free Tier với chi phí 0.00 USD ở mức tải thử nghiệm.
5. Ghi vết kiểm toán và cảnh báo tự động: Lưu trữ 100% dữ liệu truy vấn vào Amazon DynamoDB, đồng thời kích hoạt Amazon SNS gửi email thông báo khi phát hiện nguy cơ lừa đảo có xác suất từ 90% trở lên.

---

### 4. Kiến trúc giải pháp kỹ thuật

#### Sơ đồ luồng dữ liệu và tương tác dịch vụ:

![Sơ đồ Luồng Dữ liệu & Tương tác Dịch vụ](/images/architecture/serverless-system-architecture.svg)

#### Bảng danh mục dịch vụ AWS sử dụng trong dự án:

| Dịch vụ AWS | Loại hình dịch vụ | Vai trò trong kiến trúc | Lý do lựa chọn |
| :--- | :--- | :--- | :--- |
| AWS Lambda | Serverless Compute | Thực thi tính toán và chạy suy luận mô hình XGBoost. | Tự động co giãn theo yêu cầu, miễn phí 1.000.000 lượt gọi mỗi tháng, độ trễ Warm Start dưới 15 ms. |
| Amazon API Gateway v2 | Managed API Gateway | Cung cấp điểm cuối HTTPS công khai, tiếp nhận yêu cầu từ Chrome Extension. | Chuẩn HTTP API giảm 60% độ trễ và giảm 71% chi phí so với REST API, hỗ trợ cấu hình CORS trực tiếp. |
| Amazon ECR | Container Registry | Lưu trữ Docker Image chứa môi trường Python 3.12, thư viện XGBoost và tệp trọng số mô hình. | Hỗ trợ kích thước container tối đa 10 GB, giải quyết giới hạn 250 MB của tệp nén Zip trên Lambda. |
| Amazon DynamoDB | Serverless NoSQL Database | Lưu trữ nhật ký kiểm toán cho mỗi lần phân tích nội dung thư. | Độ trễ đọc ghi dưới 10 ms, miễn phí 25 GB dung lượng lưu trữ theo chính sách AWS Free Tier. |
| Amazon SNS | Pub/Sub Messaging | Phân phối thông báo sự cố qua giao thức Email. | Kiến trúc bất đồng bộ, gửi cảnh báo trong 2 giây mà không làm tăng thời gian phản hồi của API. |
| Amazon CloudWatch | Monitoring và Observability | Thu thập nhật ký thực thi, đo lường mức sử dụng bộ nhớ, độ trễ và thiết lập cảnh báo lỗi. | Tích hợp sẵn với Lambda, cung cấp bảng điều khiển trực quan và gửi thông báo khi phát sinh lỗi hệ thống. |
| AWS IAM | Identity and Access Management | Phân quyền truy cập theo nguyên tắc đặc quyền tối thiểu. | Kiểm soát chi tiết quyền hạn thực thi giữa các dịch vụ dựa trên ARN tài nguyên. |

---

### 5. Tiến độ thực hiện dự án

#### Bảng phân chia tiến độ thực hiện 12 tuần:

| Giai đoạn | Thời gian | Mục tiêu kỹ thuật | Công nghệ sử dụng | Sản phẩm hoàn thành |
| :--- | :---: | :--- | :--- | :--- |
| Khởi động và chuẩn bị dữ liệu | Tuần 1 đến tuần 2 | Nghiên cứu bài toán, xử lý dữ liệu, huấn luyện mô hình XGBoost kết hợp TruncatedSVD và thiết lập môi trường AWS. | Python, Scikit-learn, XGBoost, IAM | Tệp trọng số mô hình định dạng joblib, tài khoản AWS hoàn tất cấu hình bảo mật. |
| Đóng gói và giao tiếp mạng | Tuần 3 đến tuần 4 | Đóng gói Container Image, thiết lập kho lưu trữ Amazon ECR, cấu hình Amazon API Gateway v2 HTTP API. | Docker, Amazon ECR, Amazon API Gateway v2 | Container Image trên ECR, điểm cuối API hoạt động với phương thức POST. |
| Suy luận và lưu trữ dữ liệu | Tuần 5 đến tuần 6 | Triển khai Lambda từ Container Image, tối ưu độ trễ Warm Start dưới 15 ms, cấu hình bảng DynamoDB và tạo Topic SNS. | AWS Lambda, Amazon DynamoDB, Amazon SNS | Hàm Lambda phản hồi dữ liệu dự đoán, ghi nhật ký vào DynamoDB và gửi cảnh báo qua SNS. |
| Giao diện người dùng và giám sát | Tuần 7 đến tuần 8 | Xây dựng Chrome Extension theo chuẩn Manifest V3, kiểm thử tích hợp trên giao diện Gmail, thiết lập bảng điều khiển CloudWatch. | JavaScript, Chrome Extensions, Amazon CloudWatch | Extension hoạt động trên trình duyệt, bảng điều khiển giám sát hệ thống. |
| Mở rộng hệ thống | Tuần 9 đến tuần 12 | Nghiên cứu tích hợp AWS WAF giới hạn tần suất yêu cầu, xây dựng quy trình lưu trữ dữ liệu phân tích trên Amazon S3 và tự động hóa quy trình huấn luyện lại. | AWS WAF, Amazon S3, AWS Glue, AWS Step Functions | Tài liệu kỹ thuật hoàn chỉnh và báo cáo tổng kết dự án. |

---

### 6. Dự toán ngân sách vận hành

Hệ thống ước tính phục vụ nhu cầu kiểm tra với quy mô 100.000 yêu cầu phân tích thư điện tử mỗi tháng:

| Dịch vụ AWS | Mức sử dụng ước tính mỗi tháng | Hạn mức miễn phí AWS Free Tier | Chi phí thực tế |
| :--- | :--- | :--- | :---: |
| AWS Lambda | 100.000 yêu cầu, thời gian 15 ms, bộ nhớ 512 MB | 1.000.000 yêu cầu và 3.200.000 GB-giây mỗi tháng | 0.00 USD |
| Amazon API Gateway v2 | 100.000 HTTP requests | 1.000.000 yêu cầu miễn phí mỗi tháng trong 12 tháng đầu | 0.00 USD |
| Amazon ECR | 1 kho lưu trữ, dung lượng ảnh 210 MB | 500 MB dung lượng lưu trữ riêng tư mỗi tháng | 0.00 USD |
| Amazon DynamoDB | 100.000 lượt ghi, dung lượng 50 MB, 1 RCU và 1 WCU | 25 GB lưu trữ, 25 RCU và 25 WCU miễn phí | 0.00 USD |
| Amazon SNS | 500 thông báo email cảnh báo | 1.000 thông báo email miễn phí mỗi tháng | 0.00 USD |
| Amazon CloudWatch | 1 nhóm nhật ký 100 MB, 4 chỉ số đo lường, 2 cảnh báo | 5 GB tiếp nhận nhật ký và 10 chỉ số cảnh báo miễn phí | 0.00 USD |
| Tổng chi phí vận hành | Quy mô 100.000 yêu cầu mỗi tháng | Nằm trong hạn mức AWS Free Tier | 0.00 USD mỗi tháng |

{{% notice tip %}}
Khi quy mô lưu lượng đạt mức 1.000.000 lượt phân tích mỗi tháng, chi phí vận hành hệ thống ước tính là 1.15 USD mỗi tháng, trong đó chi phí gọi API Gateway là 1.00 USD cho một triệu yêu cầu. Chi phí này thấp hơn so với việc vận hành máy chủ EC2 t3.medium có mức phí cố định từ 35 đến 50 USD mỗi tháng.
{{% /notice %}}

---

### 7. Quản trị rủi ro và giải pháp xử lý

| Rủi ro kỹ thuật | Mức độ | Tác động | Biện pháp phòng ngừa và xử lý |
| :--- | :---: | :--- | :--- |
| Độ trễ Cold Start | Trung bình | Yêu cầu đầu tiên mất hơn 1 giây do hệ thống phải tải container image từ ECR vào môi trường thực thi. | Khởi tạo mô hình tại phạm vi biến toàn cục bên ngoài hàm xử lý để các lần thực thi tiếp theo có thời gian phản hồi dưới 15 ms. Thiết lập thời gian chờ của Lambda là 5 giây để tránh lỗi quá hạn. |
| Lỗi chính sách CORS | Cao | Trình duyệt chặn hiển thị dữ liệu do vi phạm chính sách cùng nguồn gốc giữa extension và miền API. | Cấu hình cho phép mọi nguồn gốc tại API Gateway HTTP API v2. Thêm tiêu đề Access-Control-Allow-Origin với giá trị ký tự sao trong phản hồi từ hàm Lambda. |
| Kích thước gói triển khai vượt ngưỡng cho phép | Nghiêm trọng | Các thư viện máy học như xgboost và scipy vượt quá giới hạn dung lượng 250 MB của tệp nén Zip trên Lambda. | Chuyển đổi phương thức triển khai sang Docker Container Image lưu trữ trên Amazon ECR với dung lượng hỗ trợ tối đa 10 GB. |
| Suy giảm độ chính xác mô hình theo thời gian | Trung bình | Các mẫu thư lừa đảo mới xuất hiện làm giảm hiệu quả phân loại của mô hình ban đầu. | Lưu trữ dữ liệu dự đoán vào Amazon DynamoDB để phục vụ thu thập mẫu tái huấn luyện. Xây dựng kế hoạch tích hợp quy trình huấn luyện lại với AWS Step Functions và Amazon SageMaker. |
| Tấn công từ chối dịch vụ | Thấp | Lưu lượng truy cập giả mạo gửi liên tục làm tăng số lượng xử lý và chi phí tài nguyên. | Lập phương án tích hợp AWS WAF với quy tắc giới hạn tần suất tối đa 20 yêu cầu trong 5 phút từ một địa chỉ IP. |
