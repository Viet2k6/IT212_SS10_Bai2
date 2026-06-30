## Bài 2: Tối Ưu Hóa Prompt Đặc Tả Yêu Cầu Chức Năng

**1. Nội dung Prompt tối ưu:**

```text
Act as a Senior Business Analyst.
Hãy viết tài liệu Đặc tả Yêu cầu chức năng (Functional Requirements) cho module Đăng nhập của dự án "Shop AI".
Hệ thống hiện tại đang sử dụng chuẩn bảo mật JWT (JSON Web Token) để quản lý phiên làm việc và áp dụng mô hình phân quyền RBAC (Role-Based Access Control) để quản trị quyền truy cập.
Yêu cầu bắt buộc phải đặc tả chi tiết cách hệ thống xử lý 3 trường hợp ngoại lệ sau:
1. Người dùng nhập sai mật khẩu quá 5 lần.
2. Token JWT hết hạn (expired) trong lúc phiên làm việc đang diễn ra.
3. Tài khoản người dùng ở trạng thái khóa (Inactive) nhưng cố tình thực hiện đăng nhập.
Trình bày tài liệu dưới định dạng Markdown, chia thành các phần rõ ràng: Tổng quan, Luồng chính, và Bảng xử lý ngoại lệ để dễ theo dõi.
```

**2. Tài liệu Functional Requirements do AI sinh ra (chạy thử với prompt trên):**

```markdown
# Yêu Cầu Chức Năng (Functional Requirements) - Module Đăng Nhập (Shop AI)

## 1. Tổng Quan (Overview)

Module Đăng nhập đảm nhận vai trò xác thực danh tính người dùng truy cập vào hệ thống Shop AI. Hệ thống sử dụng cơ chế JSON Web Token (JWT) để duy trì phiên làm việc không trạng thái (stateless) và áp dụng Role-Based Access Control (RBAC) để xác định quyền hạn người dùng sau khi đăng nhập thành công.

## 2. Luồng Nghiệp Vụ Chính (Main Flow)

1. Người dùng nhập thông tin `Email` và `Password` tại giao diện Đăng nhập.
2. Client mã hóa và gửi dữ liệu lên API Endpoint `/api/auth/login`.
3. Hệ thống kiểm tra tính hợp lệ của dữ liệu và đối chiếu với cơ sở dữ liệu.
4. Nếu thông tin khớp và tài khoản hợp lệ, hệ thống tạo ra một `Access Token` và `Refresh Token` chứa thông tin người dùng và danh sách Roles (Quyền hạn).
5. Hệ thống trả về token cho Client. Client lưu token an toàn và điều hướng người dùng tới trang tương ứng theo Role.

## 3. Xử Lý Các Trường Hợp Ngoại Lệ (Edge Cases)

| Tình huống ngoại lệ                               | Xử lý của hệ thống (System Response)                                                                                                                                      | Trải nghiệm người dùng (UX)                                                                                                                                                                            |
| :------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Nhập sai mật khẩu quá 5 lần**                | Hệ thống ghi nhận số lần sai. Nếu vượt quá 5 lần, chuyển trạng thái tài khoản sang khóa tạm thời trong 15-30 phút. Gửi cảnh báo bảo mật qua email của người dùng.         | Hiển thị thông báo: _"Tài khoản của bạn đã bị khóa tạm thời do nhập sai mật khẩu quá nhiều lần. Vui lòng thử lại sau 30 phút hoặc sử dụng tính năng Quên mật khẩu."_                                   |
| **2. Token JWT hết hạn lúc phiên đang diễn ra**   | Khi API trả về HTTP Status `401 Unauthorized` do Access Token hết hạn, Client tự động gọi API `/api/auth/refresh` ngầm kèm theo Refresh Token để cấp Access Token mới.    | Nếu lấy token mới thành công, người dùng **không bị gián đoạn** thao tác. Nếu Refresh Token cũng hết hạn, hệ thống buộc đăng xuất, yêu cầu đăng nhập lại, lưu trạng thái trang để quay về thao tác cũ. |
| **3. Tài khoản Inactive nhưng cố tình đăng nhập** | Dù mật khẩu chính xác, khi kiểm tra status của user là `Inactive`, hệ thống từ chối sinh token, trả về HTTP Status `403 Forbidden` và ghi log nỗ lực đăng nhập trái phép. | Hiển thị thông báo: _"Tài khoản của bạn đang bị khóa hoặc chưa được kích hoạt. Vui lòng liên hệ Bộ phận CSKH để được hỗ trợ."_                                                                         |
```
