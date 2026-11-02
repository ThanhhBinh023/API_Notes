# TỔNG HỢP KIẾN THỨC NÂNG CAO: API, REST VÀ BEST PRACTICES

Tài liệu này tổng hợp toàn diện các khái niệm, cấu trúc, tiêu chuẩn bảo mật và các nguyên tắc thực hành tốt nhất (Best Practices) khi thiết kế và làm việc với RESTful API trong môi trường thực tế.

---

## 1. Bản Chất Của API & REST

### 1.1. API (Application Programming Interface)
API là giao diện lập trình ứng dụng, cơ chế cho phép các phần mềm độc lập giao tiếp với nhau. Trong thời đại Microservices (kiến trúc vi dịch vụ) và ứng dụng di động, API đóng vai trò là "xương sống" để kết nối Front-end (Web, App) với Back-end (Database, Server logic), hoặc giữa các Server với nhau.

### 1.2. 6 Nguyên tắc (Constraints) của kiến trúc REST
Để một API được gọi là "chuẩn RESTful", nó thường phải tuân thủ 6 ràng buộc sau (5 bắt buộc, 1 tùy chọn):

1. **Client-Server (Khách - Chủ):** Sự tách biệt rạch ròi. Client quản lý giao diện, Server quản lý dữ liệu. Hai bên có thể phát triển và nâng cấp độc lập.
2. **Stateless (Phi trạng thái):** Rất quan trọng! Server không lưu giữ thông tin (session) của Client giữa các request. Mỗi Request phải chứa đủ 100% thông tin để Server hiểu và xử lý (vd: phải luôn gửi kèm Token). Giúp Server dễ dàng mở rộng (scale).
3. **Cacheable (Khả năng lưu đệm):** Các Response từ Server phải định nghĩa rõ là có được phép lưu cache hay không, giúp giảm tải cho Server và tăng tốc độ cho Client.
4. **Uniform Interface (Giao diện đồng nhất):** Mọi tài nguyên phải được định danh thống nhất qua URL. Cách thao tác với dữ liệu (thêm, sửa, xóa) phải nhất quán qua các HTTP Methods.
5. **Layered System (Hệ thống phân lớp):** Client không nhất thiết phải biết nó đang kết nối trực tiếp với Server cuối, hay đang qua các lớp trung gian như Load Balancer, API Gateway, hay Cache Server.
6. **Code on Demand (Tùy chọn):** Server có thể gửi mã thực thi (như JavaScript) về cho Client để chạy, nhưng hiếm khi được dùng trong RESTful API hiện đại.

---

## 2. Đi Sâu Vào Request & Response

### 2.1. HTTP Methods và Tính Lũy Đẳng (Idempotency)
Tính lũy đẳng có nghĩa là: Dù bạn gọi API đó 1 lần hay 100 lần, kết quả để lại trên hệ thống (trạng thái Server) vẫn hoàn toàn giống nhau. Đây là khái niệm cốt lõi khi thiết kế API.

* **GET:** Lấy dữ liệu. (Có tính lũy đẳng - Gọi bao nhiêu lần dữ liệu vẫn vậy, không bị thay đổi).
* **POST:** Tạo dữ liệu mới. (Không có tính lũy đẳng - Gọi 10 lần sẽ tạo ra 10 bản ghi mới).
* **PUT:** Cập nhật/Thay thế toàn bộ tài nguyên. (Có tính lũy đẳng - Cập nhật `name="A"` 10 lần thì tên vẫn là "A").
* **PATCH:** Cập nhật một phần (chỉ thay đổi field cần thiết).
* **DELETE:** Xóa tài nguyên. (Có tính lũy đẳng - Gọi lần 1 xóa thành công, gọi 9 lần sau trả về lỗi không tìm thấy, nhưng trạng thái server vẫn là bản ghi đó đã biến mất).

### 2.2. Chi tiết các HTTP Status Codes (Mã trạng thái)
Được chia thành 5 nhóm cơ bản:

* **2xx - Thành công (Success):**
  * `200 OK`: Request thành công.
  * `201 Created`: Tạo tài nguyên mới thành công (thường dùng cho POST).
  * `204 No Content`: Xử lý thành công nhưng không có dữ liệu trả về (thường dùng cho DELETE).
* **3xx - Điều hướng (Redirection):**
  * `301 Moved Permanently`: Tài nguyên đã chuyển sang URL mới vĩnh viễn.
* **4xx - Lỗi phía Client (Client Error):**
  * `400 Bad Request`: Client gửi sai định dạng (thiếu param, sai cấu trúc JSON).
  * `401 Unauthorized`: Lỗi xác thực (Chưa đăng nhập, thiếu Token, Token hết hạn).
  * `403 Forbidden`: Lỗi ủy quyền (Đã đăng nhập nhưng không có quyền truy cập, VD: User thường cố xóa bài của Admin).
  * `404 Not Found`: Không tìm thấy URL hoặc tài nguyên.
  * `422 Unprocessable Entity`: Dữ liệu gửi lên đúng định dạng nhưng sai logic (VD: email đã tồn tại).
* **5xx - Lỗi phía Server (Server Error):**
  * `500 Internal Server Error`: Lỗi code hoặc database sập phía máy chủ.
  * `502 Bad Gateway`: Lỗi khi đi qua proxy/gateway.
  * `503 Service Unavailable`: Server quá tải hoặc đang bảo trì.

---

## 3. Bảo Mật Nâng Cao (Security)

### 3.1. Phân phẫu cấu trúc JWT (JSON Web Token)
Khi Server cấp JWT, nó là một chuỗi gồm 3 phần ngăn cách bởi dấu chấm: `Header.Payload.Signature`

* **Header:** Chứa thông tin về loại token (JWT) và thuật toán mã hóa (VD: HS256).
* **Payload:** Chứa các thông tin thực tế muốn truyền tải (Claim) như `user_id`, `role`, và `exp` (thời gian hết hạn). *Lưu ý: Payload chỉ được mã hóa Base64 (có thể giải mã dễ dàng), TUYỆT ĐỐI không chứa mật khẩu ở đây.*
* **Signature (Chữ ký):** Được tạo ra bằng cách lấy Header + Payload băm (hash) cùng với một "Secret Key" (Chìa khóa bí mật chỉ Server giữ). Giúp Server xác minh token này có bị kẻ gian sửa đổi trên đường truyền hay không.

### 3.2. Cấu trúc OAuth 2.0
Một luồng OAuth 2.0 cơ bản bao gồm 4 thành phần (Roles):
1. **Resource Owner:** Người dùng (Chủ sở hữu dữ liệu).
2. **Client:** Ứng dụng bên thứ 3 muốn lấy dữ liệu.
3. **Authorization Server:** Máy chủ xác thực (như Server của Google). Cấp Access Token.
4. **Resource Server:** Máy chủ lưu dữ liệu (như Google Drive).

### 3.3. Các phương pháp bảo vệ API khác
* **API Key:** Dùng để định danh ứng dụng đang gọi API (thường dùng khi cung cấp API cho đối tác).
* **Rate Limiting:** Giới hạn số lượng Request trong một khoảng thời gian (VD: 100 requests/phút) để chống tấn công DDoS hoặc spam.
* **CORS (Cross-Origin Resource Sharing):** Cơ chế bảo mật trên trình duyệt, chặn các trang web ở domain này tự ý gọi API của domain khác nếu không được Server cho phép.

---

## 4. Best Practices - Nguyên tắc thiết kế API chuẩn mực

Để API dễ đọc, dễ tích hợp và chuyên nghiệp, các hệ thống lớn tuân theo các quy tắc sau:

1. **Sử dụng Danh từ số nhiều, KHÔNG dùng động từ cho URL:**
   * ✅ Đúng: `GET /users` (Lấy danh sách user)
   * ❌ Sai: `GET /getUsers`, `POST /createUser` (Vì HTTP Method đã thể hiện hành động rồi).

2. **Cấu trúc URL phân cấp rõ ràng (Hierarchy):**
   * Muốn lấy bài viết số `5` của người dùng số `12`:
   * ✅ `GET /users/12/posts/5`

3. **Versioning (Đánh phiên bản API):**
   * Luôn đặt phiên bản trong URL để dễ nâng cấp sau này mà không làm hỏng app cũ.
   * ✅ `https://api.domain.com/v1/products`

4. **Phân trang, Lọc và Sắp xếp (Pagination, Filtering & Sorting):**
   * Khi trả về danh sách, không bao giờ trả về toàn bộ hàng triệu bản ghi, phải dùng tham số (Query Params):
   * Phân trang: `GET /users?page=2&limit=20`
   * Lọc: `GET /users?role=admin&status=active`
   * Sắp xếp: `GET /users?sort=-created_at` (Sắp xếp theo ngày tạo giảm dần).

5. **Chuẩn hóa thông báo lỗi (Error Formatting):**
   * Server phải luôn trả về một cấu trúc lỗi đồng nhất để Front-end dễ xử lý.
   ```json
   {
     "error_code": "EMAIL_EXISTS",
     "message": "Địa chỉ email này đã được sử dụng.",
     "details": "email_field"
   }
   ```