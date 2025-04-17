# 🔐 Insecure Deserialization - Lý thuyết & Thực hành với PHP

## 📌 Mục tiêu
- Tìm hiểu khái niệm, cách khai thác và phòng chống lỗi Insecure Deserialization.
- Thực hành bằng cách viết một website PHP gồm 2 form: serialize và deserialize object.
- Viết mã khai thác, sau đó sửa lại mã để phòng chống lỗi.

---

## 📖 PHẦN 1: LÝ THUYẾT - Insecure Deserialization

### 🔍 Định nghĩa
**Insecure Deserialization** là lỗ hổng xảy ra khi một ứng dụng **deserialize dữ liệu không đáng tin cậy** mà không xác thực hoặc lọc đầu vào đúng cách. Điều này có thể dẫn đến:
- Thực thi mã tùy ý (Remote Code Execution - RCE)
- Ghi đè dữ liệu
- Tấn công DoS
- Tấn công truy cập trái phép

### 📈 Cách khai thác
Kẻ tấn công có thể:
- Tạo một đối tượng giả mạo (malicious object)
- Serialize đối tượng đó
- Mã hóa (base64 hoặc hex)
- Truyền chuỗi đã mã hóa vào hệ thống mục tiêu
- Nếu hệ thống deserialize mà không xác minh, đối tượng độc hại sẽ được khởi tạo và thực thi hành vi tấn công

Ví dụ: override hàm `__wakeup()` hoặc `__destruct()` để thực thi mã độc khi object được unserialize.

### 🛡️ Cách phòng chống
- **Không deserialize dữ liệu không tin cậy**
- Sử dụng các định dạng an toàn hơn như JSON, không hỗ trợ object
- Nếu bắt buộc phải dùng, thì:
  - Dùng allowlist cho class được phép deserialize
  - Vô hiệu hóa `__autoload()` trong quá trình unserialize
  - Kiểm tra và validate đầu vào
- Cập nhật PHP và thư viện lên bản vá mới nhất

🔗 Tham khảo: [PortSwigger - Insecure Deserialization](https://portswigger.net/web-security/deserialization)

---

## 💻 PHẦN 2: THỰC HÀNH - Demo và Fix Lỗi Insecure Deserialization trong PHP

### 🧪 Mô tả chương trình
- `index.php`: 
  - Form nhập thông tin: Username, Email, Năm sinh, Giới tính
  - Sau khi submit: hiển thị chuỗi base64 của object PHP
- `decode.php`: 
  - Form nhập chuỗi base64
  - Sau khi submit: unserialize object và in thông tin ra
  - **Có chứa lỗi insecure deserialization**
- `decode_fixed.php`: 
  - Phiên bản đã sửa lỗi, chỉ chấp nhận object hợp lệ (class `User`) để tránh tấn công RCE

---

## 📂 Cấu trúc file
```bash
.
├── index.php
├── decode.php          # Phiên bản dễ bị khai thác
└── decode_fixed.php    # Phiên bản đã sửa lỗi
```

---

## 🧑‍💻 Các bước thực hiện

### 1. Tạo form nhập thông tin (index.php)
- Tạo một form HTML cho phép người dùng nhập thông tin: Username, Email, Năm sinh, Giới tính.
- Khi form được submit, PHP sẽ serialize dữ liệu và trả về chuỗi base64 của đối tượng.

### 2. Tạo form nhập base64 (decode.php)
- Tạo một form HTML thứ hai cho phép người dùng nhập chuỗi base64.
- Sau khi người dùng submit, chuỗi base64 sẽ được decode và unserialize thành đối tượng PHP.
- Đoạn mã này sẽ chứa lỗ hổng insecure deserialization, cho phép kẻ tấn công gửi chuỗi base64 độc hại.

### 3. Tạo một payload khai thác
- Để khai thác lỗi, kẻ tấn công sẽ tạo một đối tượng độc hại, serialize nó, mã hóa base64 và gửi vào form thứ hai.
- Đối tượng này có thể lợi dụng các hàm như `__destruct()` hoặc `__wakeup()` để thực thi mã độc khi đối tượng bị unserialize.

### 4. Khắc phục lỗi (decode_fixed.php)
- Để khắc phục lỗi, sử dụng `unserialize()` với tùy chọn `allowed_classes`, chỉ chấp nhận những class cụ thể được deserialize.
- Thay vì deserializing bất kỳ đối tượng nào, chỉ deserialize các đối tượng hợp lệ (ví dụ: chỉ deserialize `User`).

### 5. Kiểm tra và thử nghiệm
- Sau khi khắc phục, kiểm tra lại chương trình bằng cách thử gửi các payload độc hại để chắc chắn rằng hệ thống không còn bị ảnh hưởng bởi lỗ hổng insecure deserialization.

---

## 📌 Kết luận
- Insecure Deserialization là lỗ hổng nghiêm trọng nếu không được kiểm soát đúng cách.
- Trong PHP, cần đặc biệt chú ý khi dùng `unserialize()` với dữ liệu từ client.
- Phòng chống tốt nhất là không deserialize dữ liệu không tin cậy hoặc dùng các định dạng thay thế như JSON.

---

## 📚 Tài liệu tham khảo
- https://portswigger.net/web-security/deserialization
- https://owasp.org/www-community/vulnerabilities/Deserialization_of_untrusted_data
