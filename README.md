# Bài 1 : Hệ thống Giám sát Sự cố Website Doanh nghiệp
## Phần 1: Phân tích
### 1. Lỗi sai trong lập luận của Nam
Nam cho rằng:
```text
"DNS Lookup không tìm thấy địa chỉ IP nên máy mới báo lỗi 500. Em cần đổi DNS sang 8.8.8.8 để vào lại trang web."
```
Lập luận này không chính xác.

**Vì sao sai?**

Lỗi `500 Internal Server Error` là một mã trạng thái HTTP thuộc nhóm 5xx (Server Error), nghĩa là:

- Trình duyệt đã kết nối được tới máy chủ web.
- DNS đã phân giải tên miền thành công.
- Request đã được gửi tới server.
- Server nhận request nhưng xảy ra lỗi trong quá trình xử lý.

Nếu DNS thực sự lỗi thì trình duyệt thường sẽ hiển thị các lỗi như:
```text
DNS_PROBE_FINISHED_NXDOMAIN
Server not found
This site can't be reached
ERR_NAME_NOT_RESOLVED
```
Khi đó không thể nhận được mã HTTP 500, vì mã HTTP chỉ xuất hiện sau khi đã giao tiếp thành công với máy chủ web.
### 2. So sánh các mã trạng thái HTTP
|Mã|Ý nghĩa|Phía gây lỗi|
|---|---|---|
|`200 OK`|Request thành công|Không có lỗi|
|`404 Not Found`|Tài nguyên không tồn tại|Thường do URL hoặc dữ liệu yêu cầu|
|`500 Internal Server Error`|Lỗi xử lý bên trong máy chủ|Server|
|`502 Bad Gateway`|Gateway nhận phản hồi không hợp lệ|Server|
|`503 Service Unavailable`|Server quá tải hoặc bảo trì|Server|

Kết luận:
```text
200 → Thành công
404 → Không tìm thấy tài nguyên
500 → Máy chủ xử lý bị lỗi
```
Do đó:
```text
Việc đổi DNS sang 8.8.8.8 không phải là giải pháp trực tiếp cho lỗi 500.
```
## Phần 2: Hoàn thiện
### Mô tả luồng dữ liệu từ DNS Lookup đến Server Response
```text
Người dùng nhập:

https://lms.ptit.edu.vn
          │
          ▼
1. DNS Lookup
   Hỏi DNS server:
   "IP của lms.ptit.edu.vn là gì?"
          │
          ▼
2. DNS trả về IP
   Ví dụ:
   203.xxx.xxx.xxx
          │
          ▼
3. TCP Handshake
   SYN
   SYN-ACK
   ACK
          │
          ▼
4. TLS Handshake (HTTPS)
   Thiết lập kết nối bảo mật
          │
          ▼
5. Browser gửi HTTP Request

   GET /
   Host: lms.ptit.edu.vn

          │
          ▼
6. Web Server nhận Request
          │
          ▼
7. Ứng dụng xử lý
   - Truy vấn Database
   - Kiểm tra Session
   - Xử lý Business Logic
          │
          ▼
8. Phát sinh lỗi nội bộ
   Ví dụ:
   - SQL Error
   - Exception chưa bắt
   - Thiếu file cấu hình
   - Lỗi kết nối Database
          │
          ▼
9. Server trả về:

HTTP/1.1 500 Internal Server Error

          │
          ▼
10. Browser hiển thị
    "500 Internal Server Error"
```
**Vì sao Nam nhận được mã 500?**

Nếu trình duyệt hiển thị:
```text
500 Internal Server Error
```
thì điều này chứng minh rằng:

✅ DNS đã hoạt động.

✅ Tên miền đã được phân giải.

✅ Đã kết nối tới máy chủ.

✅ HTTP Request đã đến máy chủ.

❌ Máy chủ xử lý thất bại.

Do đó lỗi nằm ở:
```text
Server Side
```
không phải:
```text
Client Side
DNS
Mạng Internet của Nam
```
