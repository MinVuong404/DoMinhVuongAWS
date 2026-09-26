---
title: "Chrome Extension & Gmail"
date: 2026-08-25
weight: 6
chapter: false
pre: " <b> 4.6. </b> "
---

# 4.6. Cài đặt Chrome Extension và kiểm thử trên Gmail

### 4.6.1. Cấu trúc tiện ích mở rộng Chrome Extension

Mã nguồn client được tổ chức theo tiêu chuẩn Manifest V3 của Google Chrome:

#### Danh mục tệp mã nguồn trong thư mục gmail_extension:

| Tên tệp hoặc thư mục | Định dạng | Vai trò và chức năng kỹ thuật |
| :--- | :--- | :--- |
| `manifest.json` | Cấu hình JSON | Khai báo quyền hạn activeTab, storage, host permissions và tài nguyên tiện ích. |
| `content.js` | JavaScript | Lắng nghe cấu trúc DOM của Gmail, bóc tách nội dung thư và gửi yêu cầu phân tích tới API Gateway. |
| `background.js` | Service Worker | Điều phối các thông điệp bất đồng bộ và quản lý trạng thái phiên làm việc. |
| `popup.html` | Giao diện HTML | Hiển thị thông tin khi người dùng mở biểu tượng tiện ích trên thanh công cụ trình duyệt. |
| `popup.js` | JavaScript | Xử lý tương tác trên giao diện popup và lưu trữ cấu hình điểm cuối API Gateway. |
| `styles.css` | Tệp CSS | Định dạng giao diện nút bấm kiểm tra và bảng thông báo kết quả trên trang Gmail. |
| `icons/` | Thư mục hình ảnh | Chứa các biểu tượng kích thước chuẩn 16x16, 48x48 và 128x128 pixel. |

#### Đoạn mã xử lý tương tác DOM trong content.js
```javascript
// Khởi tạo nút kiểm tra bảo mật trong giao diện đọc thư của Gmail
function injectScanButton() {
  const toolbar = document.querySelector('.G-Ni.J-J5-Ji');
  if (!toolbar || document.getElementById('aws-phishing-scan-btn')) return;

  const btn = document.createElement('button');
  btn.id = 'aws-phishing-scan-btn';
  btn.className = 'aws-scan-button';
  btn.innerHTML = 'Quét Phishing với AWS AI';
  
  btn.onclick = async () => {
    btn.innerHTML = 'Đang phân tích...';
    btn.disabled = true;
    
    const emailBody = document.querySelector('.a3s.aiL')?.innerText || '';
    try {
      const res = await fetch('https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email_text: emailBody })
      });
      const data = await res.json();
      displayResultBadge(data);
    } catch (err) {
      alert('Lỗi kết nối tới AWS Backend: ' + err.message);
    } finally {
      btn.innerHTML = 'Quét Phishing với AWS AI';
      btn.disabled = false;
    }
  };

  toolbar.appendChild(btn);
}

// Giám sát liên tục các thay đổi DOM trong Gmail bằng MutationObserver
const observer = new MutationObserver(() => injectScanButton());
observer.observe(document.body, { childList: true, subtree: true });
```

---

### 4.6.2. Hướng dẫn cài đặt tiện ích vào trình duyệt Chrome

1. Mở trình duyệt Chrome, truy cập địa chỉ chrome://extensions/.
2. Bật tùy chọn Chế độ dành cho nhà phát triển ở góc trên bên phải màn hình.
3. Nhấp vào nút Tải tiện ích đã giải nén ở góc trên bên trái.
4. Chọn thư mục gmail_extension tại đường dẫn của dự án.
5. Kiểm tra tiện ích AWS Serverless Phishing Detector for Gmail xuất hiện trong danh sách và ở trạng thái kích hoạt.

---

### 4.6.3. Kiểm thử thực tế trên giao diện Gmail

Thực hiện kiểm tra trực tiếp trên các mẫu thư tại địa chỉ mail.google.com:

#### Kịch bản 1: Email thông báo công việc thông thường
* Tiêu đề: Thông báo lịch họp bảo vệ đồ án tốt nghiệp Khoa Công nghệ Thông tin
* Nội dung: Kính gửi sinh viên Đỗ Minh Vương, lịch bảo vệ đồ án của lớp 68CNMHT dự kiến diễn ra vào sáng thứ Bảy tại phòng Hội thảo 302-A1.
* Thao tác: Nhấp nút Quét Phishing với AWS AI.
* Kết quả hiển thị:
  * Nhãn kết quả: Safe màu xanh lá cây.
  * Xác suất lừa đảo: 2.15%.
  * Thời gian suy luận: 11.20 ms.
  * Hành động cảnh báo: Không kích hoạt SNS do giá trị xác suất dưới ngưỡng 90%.

#### Kịch bản 2: Email lừa đảo mạo danh ngân hàng
* Tiêu đề: Khẩn cấp, tài khoản Vietcombank của bạn đang bị khóa do phát hiện giao dịch đáng ngờ
* Nội dung: Hệ thống phát hiện giao dịch rút 30.000.000 VNĐ tại TP.HCM. Nếu không phải bạn thực hiện, vui lòng truy cập đường dẫn https://vietcombank-xac-thuc-otp.xyz/security để nhập mã OTP hủy giao dịch trong 15 phút.
* Thao tác: Nhấp nút Quét Phishing với AWS AI.
* Kết quả hiển thị:
  * Nhãn kết quả: CRITICAL PHISHING ALERT màu đỏ.
  * Xác suất lừa đảo: 99.45%.
  * Thời gian suy luận: 12.80 ms.
  * Hành động cảnh báo: Hệ thống gửi thông báo qua SNS tới hòm thư quản trị viên sau 1.3 giây.
  * Hành động lưu trữ: Bản ghi kiểm toán được tạo trong bảng DynamoDB với mã yêu cầu UUID tương ứng.

---

### Bảng kết quả thực nghiệm 20 mẫu email

| STT | Loại kịch bản email | Số lượng mẫu | Phân loại đúng | Phân loại sai | Độ chính xác | Thời gian phản hồi trung bình | Số lượng cảnh báo SNS phát sinh |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| 1 | Email công việc và thông báo trường học | 10 | 10 | 0 | 100% | 185 ms | 0 |
| 2 | Email giả mạo tổ chức tài chính và yêu cầu mã OTP | 6 | 6 | 0 | 100% | 192 ms | 6 |
| 3 | Email lừa đảo trúng thưởng hoặc tuyển dụng giả mạo | 4 | 4 | 0 | 100% | 188 ms | 4 |
| Tổng | Toàn bộ các mẫu thử nghiệm | 20 | 20 | 0 | 100.0% | 188.3 ms | 10 trên 10 mẫu Phishing |
