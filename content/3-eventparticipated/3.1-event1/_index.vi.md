---
title: "Sự kiện 1: AWS Community Meetup"
date: 2026-07-25
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Báo cáo thu hoạch sự kiện AWS Vietnam Community Meetup: AI Revolution và Open Claw

### 1. Thông tin chung về sự kiện
* Tên sự kiện: AWS Vietnam Community Meetup Hà Nội
* Chủ đề chính: AI Revolution và Open Claw
* Thời gian tổ chức: Từ 08:30 đến 12:00, Thứ Bảy, ngày 25/07/2026
* Địa điểm tổ chức: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, Số 36 Cát Linh, Ô Chợ Dừa, Đống Đa, TP. Hà Nội
* Đơn vị tổ chức: AWS Vietnam User Group, AWS First Cloud Journey AI, AWS Student Builder Group tại UTC phối hợp cùng AWS Việt Nam
* Vai trò tham gia: Người tham dự và trao đổi thảo luận kỹ thuật

<div style="text-align: center; margin: 25px 0;">
  <img src="/images/3-eventparticipated/event1/aws_community_meetup_poster.jpg" alt="Poster sự kiện AWS Vietnam Community Meetup - AI Revolution và Open Claw tại Văn phòng AWS Hà Nội" style="border-radius: 8px; box-shadow: 0 4px 16px rgba(0,0,0,0.15); max-width: 85%; height: auto; border: 1px solid #E2E8F0;" />
  <p style="font-style: italic; color: #666; margin-top: 10px;">Hình 3.1: Poster chính thức sự kiện AWS Vietnam Community Meetup tại Văn phòng AWS Hà Nội</p>
</div>

---

### 2. Mục đích và nội dung tổng quan
* Tìm hiểu sự chuyển dịch từ các mô hình ngôn ngữ lớn sang mô hình AI Agents có khả năng lập kế hoạch và gọi công cụ bên ngoài.
* Tiếp nhận kinh nghiệm triển khai thực tế từ các kỹ sư và kiến trúc sư giải pháp của AWS.
* Phân tích phương pháp gắn kết giữa ứng dụng AI và hiệu quả đầu tư kinh doanh.
* Trao đổi kỹ thuật với các kỹ sư điện toán đám mây và cộng đồng sinh viên công nghệ thông tin.

---

### 3. Danh sách diễn giả và khách mời chuyên môn
* Hồ Việt Anh và Phong Phạm: Đại diện điều phối cộng đồng AWS User Group Việt Nam.
* Tuấn Vũ: Kỹ sư phần mềm, chuyên gia nghiên cứu AI Agent mã nguồn mở.
* Nguyễn Thu và Nam La: Chuyên gia tư vấn giải pháp chuyển đổi số và AI cho doanh nghiệp.
* Henry Đức Bùi: Kỹ sư giải pháp, chuyên sâu về phương pháp phát triển sản phẩm tinh gọn.

---

### 4. Nội dung chuyên môn tại sự kiện

#### 4.1. Thông tin hoạt động cộng đồng
* Diễn giả: Hồ Việt Anh và Phong Phạm.
* Nội dung: Tổng kết các hoạt động của cộng đồng AWS Việt Nam, kế hoạch các buổi hội thảo kỹ thuật tiếp theo và chương trình đào tạo điện toán đám mây First Cloud AI Journey.

#### 4.2. OpenClaw, xu hướng và thực tiễn ứng dụng AI Agent mã nguồn mở
* Diễn giả: Tuấn Vũ.
* Nội dung:
  * Trình bày kiến trúc của framework AI Agent mã nguồn mở OpenClaw gồm các thành phần lập kế hoạch, lưu trữ ngữ cảnh và giao tiếp với API bên ngoài.
  * Phương pháp tối ưu hóa chi phí hạ tầng khi vận hành các tác vụ tự động hóa liên tục trên nền tảng AWS.

#### 4.3. Chuyển dịch từ xu hướng AI sang giá trị kinh doanh
* Diễn giả: Nguyễn Thu và Nam La.
* Nội dung:
  * Phân tích mối liên hệ giữa việc áp dụng công nghệ AI và hiệu quả kinh tế thực tế của doanh nghiệp.
  * Đánh giá việc lựa chọn mô hình: Tránh sử dụng các mô hình ngôn ngữ lớn có chi phí tính toán cao cho các tác vụ phân loại nhị phân đơn giản. Việc kết hợp các mô hình học máy truyền thống như XGBoost hoặc SVM trên hạ tầng Serverless mang lại hiệu quả cao hơn về mặt chi phí và tốc độ xử lý.

#### 4.4. Phương pháp phát triển sản phẩm với AI
* Diễn giả: Henry Đức Bùi.
* Nội dung:
  * Phân tích nguyên tắc ứng dụng AI làm công cụ hỗ trợ tăng năng suất, đồng thời duy trì việc kiểm soát kiến trúc hệ thống, an toàn thông tin và chất lượng mã nguồn.

#### 4.5. Trao đổi kỹ thuật và định hướng nghề nghiệp
* Thảo luận về lộ trình phát triển chuyên môn của vị trí Cloud Solutions Architect và các kỹ năng kỹ thuật cần thiết trong môi trường doanh nghiệp.

---

### 5. Bài học áp dụng vào dự án thực tập

#### 5.1. Tư duy kiến trúc hệ thống
* Lựa chọn công nghệ phù hợp với yêu cầu bài toán: Đối với bài toán phân loại email lừa đảo, việc sử dụng mô hình học máy XGBoost kết hợp TruncatedSVD có dung lượng 38 MB triển khai trên AWS Lambda mang lại thời gian xử lý 12 ms và chi phí nằm trong hạn mức miễn phí, thay vì sử dụng các mô hình ngôn ngữ lớn đòi hỏi GPU tốn kém.
* Nắm vững kiến trúc nền tảng: Tự cấu hình tệp Dockerfile, phân quyền IAM theo nguyên tắc đặc quyền tối thiểu và kiểm soát kết nối mạng giữa các dịch vụ.

#### 5.2. Ứng dụng cụ thể trong dự án
* Thiết kế API Gateway v2 chuẩn RESTful để tiếp nhận yêu cầu suy luận từ các nền tảng khách hàng khác nhau.
* Thiết lập chính sách CORS và phân quyền IAM nhằm đảm bảo an toàn truy cập dữ liệu giữa trình duyệt và hạ tầng AWS.

---

### 6. Đánh giá kết quả tham gia
Sự kiện cung cấp các phân tích thực tế về bài toán tối ưu hóa chi phí hạ tầng khi ứng dụng học máy trong doanh nghiệp. Các nội dung về lựa chọn mô hình tính toán tinh gọn được áp dụng trực tiếp vào quá trình thiết kế hệ thống phát hiện email lừa đảo trong dự án thực tập.
