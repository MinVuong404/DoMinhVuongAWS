---
title: "Nhật ký Tuần 3"
date: 2026-08-17
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Nhật ký Tuần 3: Kiến trúc giao tiếp mạng và cổng Amazon API Gateway v2

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 17/08/2026 đến 23/08/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Nghiên cứu mô hình mạng Amazon Virtual Private Cloud gồm Subnet, Route Tables và Internet Gateway.
  2. So sánh hai loại hình giao tiếp của API Gateway là REST API và HTTP API.
  3. Thiết kế chuẩn giao tiếp RESTful API cho hệ thống, quy định cấu trúc Schema JSON cho dữ liệu gửi lên và kết quả trả về.
  4. Cấu hình cơ chế chia sẻ tài nguyên giữa các nguồn gốc CORS nhằm xử lý yêu cầu gửi từ tiện ích mở rộng trên miền mail.google.com.
  5. Thiết lập cơ chế tích hợp Lambda Proxy Integration theo định dạng Payload Format 2.0.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Tìm hiểu cấu trúc Amazon VPC, phân chia dải mạng 10.0.0.0/16, phân định Public Subnets và Private Subnets. Đánh giá tác động của Hyperplane ENI đối với độ trễ Cold Start của Lambda khi gán vào mạng ảo. | 17/08/2026 | 17/08/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)<br>[Improved VPC Networking for AWS Lambda](https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/) | Quyết định triển khai Lambda ở chế độ mặc định ngoài VPC nhằm giảm độ trễ khởi động do dịch vụ không cần truy cập vào tài nguyên cơ sở dữ liệu nội bộ. |
| Thứ 3 | So sánh thông số giữa REST API và HTTP API v2 trên API Gateway: HTTP API giảm khoảng 60% phụ phí độ trễ và có mức giá 1.00 USD cho một triệu yêu cầu so với 3.50 USD của REST API. | 18/08/2026 | 18/08/2026 | [Choosing Between REST APIs and HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html) | Lựa chọn chuẩn HTTP API v2 làm cổng tiếp nhận lưu lượng từ tiện ích mở rộng Chrome Extension. |
| Thứ 4 | Thiết kế tài liệu đặc tả giao tiếp JSON giữa Extension và API Gateway. Viết quy chuẩn điểm cuối API với phương thức POST tại đường dẫn /predict. | 19/08/2026 | 19/08/2026 | [OpenAPI 3.0 Specification](https://swagger.io/specification/)<br>[JSON Schema Core](https://json-schema.org/) | Hoàn thành tài liệu quy định định dạng dữ liệu đầu vào và các mã trạng thái phản hồi gồm 200, 400 và 500. |
| Thứ 5 | Cấu hình tham số CORS trên API Gateway: Cho phép các phương thức POST và OPTIONS, khai báo các tiêu đề hợp lệ Content-Type và Authorization. Kiểm tra yêu cầu Preflight OPTIONS bằng cURL. | 20/08/2026 | 20/08/2026 | [Configuring CORS for an HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html)<br>[MDN Web Docs: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) | Tiêu đề Access-Control-Allow-Origin được trả về chính xác trong phản hồi từ API Gateway. |
| Thứ 6 | Viết hàm xử lý Python bóc tách dữ liệu chuỗi ký tự từ trường event body. Bổ sung đoạn mã xử lý trường hợp dữ liệu bị mã hóa theo chuẩn Base64. | 21/08/2026 | 21/08/2026 | [Working with AWS Lambda Proxy Integrations](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html) | Hàm Lambda tiếp nhận và giải mã chính xác dữ liệu văn bản gửi lên ở định dạng chuỗi thô hoặc tệp JSON. |
| Thứ 7 và Chủ nhật | Kiểm thử gửi yêu cầu từ Postman và cURL để đo lường thời gian định tuyến của API Gateway. Thời gian chuyển tiếp yêu cầu từ API Gateway tới Lambda ghi nhận trung bình 8 ms. | 22/08/2026 | 23/08/2026 | [Postman API Platform](https://www.postman.com/)<br>[cURL Documentation](https://curl.se/docs/) | Điểm cuối API định tuyến ổn định với các mẫu thử nghiệm. |

---

### 3. Thao tác kỹ thuật và đặc tả giao thức dữ liệu

#### 3.1. Cấu trúc yêu cầu và phản hồi tại điểm cuối POST /predict
Dữ liệu gửi từ Client lên API Gateway:
```json
{
  "email_text": "CẢNH BÁO KHẨN CẤP: Tài khoản ngân hàng số của bạn đã bị khóa do nghi ngờ giao dịch gian lận. Vui lòng bấm vào đường dẫn https://secure-bank-login.xyz/verify để xác thực danh tính ngay lập tức trong vòng 24h!"
}
```
Dữ liệu phản hồi từ Lambda qua API Gateway về Client:
```json
{
  "request_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "prediction": "Phishing",
  "confidence": 0.9842,
  "confidence_percent": "98.42%",
  "risk_level": "CRITICAL",
  "inference_latency_ms": 11.45,
  "timestamp": "2026-08-20T10:15:30.124Z"
}
```

#### 3.2. Kiểm tra phản hồi CORS Preflight bằng cURL
```bash
curl -X OPTIONS https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
  -H "Origin: https://mail.google.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -i
```
Tiêu đề phản hồi ghi nhận:
```http
HTTP/2 204
date: Thu, 20 Aug 2026 10:15:30 GMT
access-control-allow-origin: *
access-control-allow-methods: POST,OPTIONS
access-control-allow-headers: content-type
access-control-max-age: 300
```

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Lỗi CORS policy do thiếu tiêu đề Access-Control-Allow-Origin khi gọi API từ Chrome Extension.
  * Hiện tượng: Khi tiện ích mở rộng trên trang mail.google.com gửi yêu cầu fetch đến API Gateway, trình duyệt Chrome chặn phản hồi và thông báo lỗi vi phạm chính sách cùng nguồn gốc.
  * Nguyên nhân: API Gateway HTTP API v2 chưa thiết lập cấu hình tự động cho các yêu cầu Preflight OPTIONS. Khi trình duyệt gửi yêu cầu POST có tiêu đề Content-Type application/json, hệ thống sẽ gửi trước một gói tin OPTIONS để kiểm tra quyền truy cập. Do thiếu cấu hình, yêu cầu bị chặn tại tầng mạng.
  * Giải pháp xử lý: Cấu hình bảng thông số CORS trên API Gateway với nguồn gốc cho phép là ký tự sao, tiêu đề cho phép gồm Content-Type, X-Amz-Date, Authorization, X-Api-Key và các phương thức POST, OPTIONS. Đồng thời, cấu hình mã nguồn Lambda luôn trả về các tiêu đề CORS tương ứng trong đối tượng phản hồi:
    ```python
    headers = {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "POST,OPTIONS"
    }
    ```
    Sau khi đồng bộ cấu hình ở cả API Gateway và mã nguồn hàm Lambda, trình duyệt tiếp nhận dữ liệu bình thường.

---

### 5. Kết quả hoàn thành

1. Xác định mô hình mạng phù hợp cho hàm Lambda không phụ thuộc vào VPC nhằm giảm thời gian khởi động.
2. Hoàn thành tài liệu đặc tả giao tiếp dữ liệu cho điểm cuối POST /predict trên Amazon API Gateway v2.
3. Thiết lập chính sách CORS trên API Gateway và hàm Lambda, cho phép tiện ích trình duyệt Chrome gửi và nhận dữ liệu kiểm tra nội dung thư.
