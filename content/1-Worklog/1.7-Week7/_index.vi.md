---
title: "Nhật ký Tuần 7"
date: 2026-09-14
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Nhật ký Tuần 7: Phát triển tiện ích Chrome Extension và kiểm thử toàn trình trên Gmail

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 14/09/2026 đến 20/09/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Phát triển tiện ích mở rộng Chrome Extension theo tiêu chuẩn Manifest V3 gồm các tệp manifest.json, content.js, background.js, popup.html, popup.js và styles.css.
  2. Bóc tách nội dung email từ cây cấu trúc Gmail DOM bằng các bộ chọn CSS gồm .a3s.aiL, .hP và .gD.
  3. Tích hợp nút kiểm tra nội dung thư vào thanh công cụ đọc thư của giao diện Gmail.
  4. Hiển thị nhãn cảnh báo phân cấp rủi ro theo màu sắc gồm An toàn, Nghi vấn hoặc Nguy hiểm kèm giá trị phần trăm độ tin cậy và thời gian suy luận.
  5. Thực hiện kiểm thử toàn trình trên 20 mẫu email gồm 10 thư công việc thông thường và 10 thư có nội dung giả mạo lừa đảo.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Soạn thảo tệp manifest.json theo chuẩn Manifest V3: Khai báo quyền activeTab, storage, cấu hình quyền truy cập trang mail.google.com và điểm cuối API Gateway. Thiết kế bộ biểu tượng kích thước 16x16, 48x48 và 128x128. | 14/09/2026 | 14/09/2026 | [Chrome Extension Manifest V3 Migration](https://developer.chrome.com/docs/extensions/mv3/intro/) | Cài đặt tiện ích vào trình duyệt Chrome ở chế độ dành cho nhà phát triển thành công. |
| Thứ 3 | Viết mã nguồn content.js: Sử dụng MutationObserver để theo dõi sự kiện mở thư trong Gmail. Trích xuất tiêu đề qua bộ chọn .hP, địa chỉ người gửi qua .gD và nội dung phần thân qua .a3s.aiL. | 15/09/2026 | 15/09/2026 | [MutationObserver Web API](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)<br>[Gmail DOM Structure Analysis](https://github.com/) | Thuật toán trích xuất lấy chính xác nội dung thư, loại bỏ các đoạn mã định dạng thừa. |
| Thứ 4 | Xây dựng giao diện nút bấm và khung hiển thị kết quả bằng CSS. Tích hợp hàm fetch gửi yêu cầu bất đồng bộ đến điểm cuối HTTPS của API Gateway. | 16/09/2026 | 16/09/2026 | [Fetch API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) | Tổng thời gian từ khi bấm nút đến khi kết quả hiển thị trên màn hình ghi nhận từ 180 đến 220 ms. |
| Thứ 5 | Kiểm thử với 10 mẫu email an toàn gồm thư mời họp, thông báo nội bộ và sao kê tài khoản ngân hàng chính thức. Ghi nhận kết quả: Toàn bộ 10 mẫu được phân loại là Safe với độ tin cậy trên 94%. | 17/09/2026 | 17/09/2026 | [Software Testing Standards](https://www.iso.org/standard/64835.html) | Tỷ lệ nhận diện nhầm dương tính giả đạt 0% trên tập dữ liệu an toàn. |
| Thứ 6 | Kiểm thử với 10 mẫu email lừa đảo gồm thư giả mạo khóa thẻ, thông báo trúng thưởng và yêu cầu xác minh mật khẩu. Ghi nhận kết quả: Cả 10 mẫu được nhận diện là Phishing với độ tin cậy từ 91.2% đến 99.1%. | 18/09/2026 | 18/09/2026 | [Anti-Phishing Working Group (APWG)](https://apwg.org/) | Hệ thống kích hoạt dịch vụ SNS gửi email cảnh báo tới hòm thư quản trị viên cho các trường hợp vượt ngưỡng 90%. |
| Thứ 7 và Chủ nhật | Lưu trữ hình ảnh và số liệu kiểm thử thực tế phục vụ báo cáo. Điều chỉnh CSS tương thích với chế độ nền tối Dark Mode trên Gmail. | 19/09/2026 | 20/09/2026 | [Responsive CSS & Theme Compatibility](https://web.dev/responsive-web-design-basics/) | Tiện ích mở rộng hiển thị ổn định trên các chế độ giao diện khác nhau của trình duyệt. |

---

### 3. Thao tác kỹ thuật và mã nguồn Chrome Extension

#### 3.1. Cấu hình tệp manifest.json chuẩn Manifest V3
```json
{
  "manifest_version": 3,
  "name": "AWS Serverless Phishing Detector for Gmail",
  "version": "1.0.0",
  "description": "Phát hiện và cảnh báo email lừa đảo thời gian thực bằng Machine Learning trên AWS Serverless",
  "permissions": ["activeTab", "storage"],
  "host_permissions": [
    "https://mail.google.com/*",
    "https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/*"
  ],
  "content_scripts": [
    {
      "matches": ["https://mail.google.com/*"],
      "js": ["content.js"],
      "css": ["styles.css"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  }
}
```

#### 3.2. Đoạn mã trích xuất DOM Gmail và gửi yêu cầu trong content.js
```javascript
// Bóc tách văn bản email từ DOM của Gmail
function extractEmailContent() {
  const bodyElement = document.querySelector('.a3s.aiL');
  if (!bodyElement) return null;
  return bodyElement.innerText.trim();
}

// Gửi nội dung tới API Gateway v2
async function scanEmailForPhishing(emailText) {
  const endpoint = 'https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict';
  const response = await fetch(endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email_text: emailText })
  });
  return await response.json();
}
```

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Trình duyệt không lấy được nội dung email khi người dùng chuyển đổi qua lại giữa các thư.
  * Hiện tượng: Khi mở thư đầu tiên thì nút kiểm tra hoạt động, nhưng khi chuyển sang thư tiếp theo trong hộp thư, hàm document.querySelector trả về giá trị rỗng hoặc giữ nguyên nội dung của thư cũ trước đó.
  * Nguyên nhân: Gmail là ứng dụng trang đơn Single-Page Application. Khi chuyển thư, giao diện không tải lại toàn bộ trang mà cập nhật động cây DOM qua AJAX. Việc sử dụng sự kiện tải trang tĩnh window.onload không theo dõi được các thay đổi thành phần con sau đó.
  * Giải pháp xử lý: Sử dụng đối tượng MutationObserver để theo dõi liên tục các thay đổi cấu trúc bên trong khung hiển thị thư div role main. Khi phát hiện phần tử có lớp .a3s.aiL xuất hiện, tiện ích tự động gắn nút kiểm tra tương ứng và xóa dữ liệu kết quả trước đó.

---

### 5. Kết quả hoàn thành

1. Hoàn thành tiện ích mở rộng Chrome Extension tương thích với giao diện Gmail theo chuẩn Manifest V3.
2. Kiểm thử thực tế trên 20 mẫu email cho kết quả phân loại phù hợp với nhãn thực tế của dữ liệu.
3. Tổng thời gian xử lý toàn trình từ thao tác người dùng đến khi hiển thị kết quả trên giao diện đạt dưới 200 ms, bao gồm độ trễ truyền dẫn mạng tới khu vực Singapore và thời gian suy luận của mô hình.
