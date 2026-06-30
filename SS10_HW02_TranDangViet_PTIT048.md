# Bài 2: Tối Ưu Hóa Prompt Đặc Tả Yêu Cầu Chức Năng

## 1. Prompt tối ưu

```text
# Vai trò (Role)
Act as a Senior System Analyst có nhiều kinh nghiệm xây dựng tài liệu SRS và Functional Requirements cho các hệ thống thương mại điện tử sử dụng Spring Boot, JWT Authentication và RBAC.

# Mục tiêu (Objective)
Hãy xây dựng tài liệu Functional Requirements cho module Authentication (Đăng nhập) của dự án Shop AI. Tài liệu cần mô tả đầy đủ các yêu cầu chức năng, luồng xử lý và các trường hợp ngoại lệ để phục vụ cho việc thiết kế, phát triển và kiểm thử hệ thống.

# Ngữ cảnh (Context)
Hệ thống Shop AI sử dụng:
- JWT (JSON Web Token) để xác thực người dùng.
- RBAC (Role-Based Access Control) để phân quyền.
- Người dùng đăng nhập bằng email và mật khẩu.
- Sau khi đăng nhập thành công, hệ thống sinh Access Token để người dùng truy cập các chức năng theo đúng vai trò.
- Các vai trò gồm: Admin, Staff và Customer.

# Ràng buộc (Constraints)
Tài liệu phải mô tả đầy đủ:
- Điều kiện trước khi thực hiện đăng nhập.
- Luồng xử lý chính của chức năng đăng nhập.
- Điều kiện sau khi đăng nhập thành công.
- Quy tắc xác thực thông tin đăng nhập.
- Quy tắc phân quyền bằng RBAC.
- Thông báo lỗi tương ứng cho từng trường hợp.

Bắt buộc phải đặc tả cách hệ thống xử lý các trường hợp ngoại lệ sau:
1. Người dùng nhập sai mật khẩu quá 5 lần.
2. Token JWT hết hạn (Expired) trong khi phiên làm việc đang diễn ra.
3. Tài khoản người dùng ở trạng thái Inactive nhưng vẫn cố gắng đăng nhập.

Không bỏ sót bất kỳ trường hợp ngoại lệ nào.

# Định dạng (Format)
Xuất kết quả bằng Markdown với các mục:
1. Functional Overview
2. Actors
3. Preconditions
4. Functional Requirements
5. Main Flow
6. Business Rules
7. Exception Handling
8. Postconditions
9. Acceptance Criteria
```

---

## 2. Functional Requirements do AI sinh ra

# Tài Liệu Đặc Tả Yêu Cầu Chức Năng (SRS) - Module Authentication

**Dự án:** Shop AI
**Module:** Authentication (Đăng nhập)
**Công nghệ tích hợp:** Spring Boot, JWT, RBAC

---

## 1. Functional Overview (Tổng Quan Chức Năng)

Module Authentication đóng vai trò là "người gác cổng" của toàn bộ hệ thống Shop AI, chịu trách nhiệm xác thực danh tính người dùng thông qua Email và Mật khẩu. Khi xác thực thành công, hệ thống sử dụng cơ chế bảo mật **JWT (JSON Web Token)** để cấp phát phiên làm việc phi trạng thái (stateless). Đồng thời, hệ thống ứng dụng mô hình **RBAC (Role-Based Access Control)** để mã hóa phân quyền vào Token, nhằm kiểm soát các chức năng và dữ liệu người dùng được phép truy cập theo từng vai trò.

## 2. Actors (Tác Nhân)

Chức năng này phục vụ cho 3 nhóm đối tượng chính đã được định nghĩa trong hệ thống RBAC:

- **Customer (Khách hàng):** Truy cập frontend/app để mua sắm, quản lý đơn hàng cá nhân.
- **Staff (Nhân viên):** Truy cập portal nội bộ để xử lý đơn hàng, chăm sóc khách hàng, quản lý kho cơ bản.
- **Admin (Quản trị viên):** Quyền cao nhất, quản trị toàn bộ hệ thống, người dùng, phân quyền, và báo cáo doanh thu.

## 3. Preconditions (Điều Kiện Tiên Quyết)

- Người dùng phải sở hữu một tài khoản hợp lệ đã được đăng ký và lưu trữ trong cơ sở dữ liệu của hệ thống.
- Mật khẩu của người dùng trong cơ sở dữ liệu phải được băm (hash) an toàn (ví dụ: sử dụng Bcrypt).
- Dịch vụ Database và Authentication API phải ở trạng thái hoạt động (Up & Running).

## 4. Functional Requirements (Yêu Cầu Chức Năng)

| Mã Yêu Cầu (FR-ID) | Tên Yêu Cầu         | Mô tả chi tiết                                                                                                                       |
| :----------------- | :------------------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| **FR-AUTH-01**     | Xác thực người dùng | Hệ thống cho phép người dùng nhập Email và Mật khẩu để đăng nhập.                                                                    |
| **FR-AUTH-02**     | Cấp phát Token      | Sau khi xác thực thành công, hệ thống sinh ra Access Token và Refresh Token chuẩn JWT trả về cho Client.                             |
| **FR-AUTH-03**     | Tích hợp RBAC       | JWT payload phải chứa thông tin về danh sách Roles (Admin, Staff, Customer) của user để Client điều hướng và API Gateway phân quyền. |
| **FR-AUTH-04**     | Theo dõi đăng nhập  | Hệ thống phải ghi log (audit) địa chỉ IP, thiết bị và thời gian mỗi lần đăng nhập thành công/thất bại.                               |

## 5. Main Flow (Luồng Xử Lý Chính)

Đây là luồng kịch bản người dùng đăng nhập thành công (Happy Path):

1. Người dùng truy cập trang Đăng nhập và nhập `Email`, `Password`.
2. Người dùng nhấn nút "Đăng nhập".
3. Client (Frontend/App) validate định dạng sơ bộ, mã hóa payload và gửi HTTP POST request đến endpoint `/api/auth/login`.
4. API Server nhận request, tìm kiếm người dùng trong cơ sở dữ liệu dựa vào `Email`.
5. Server sử dụng thuật toán băm để so sánh `Password` được gửi lên với mật khẩu đã băm trong cơ sở dữ liệu.
6. Nếu mật khẩu khớp và tài khoản hợp lệ (Active), Server thiết lập lại biến đếm số lần sai mật khẩu về 0.
7. Server khởi tạo **Access Token** (chứa UserID, Email, Roles, ExpiryTime) và **Refresh Token**.
8. Server trả HTTP 200 OK kèm bộ Token về cho Client.
9. Client lưu trữ token an toàn (ví dụ: HttpOnly Cookie hoặc Secure Storage) và điều hướng người dùng tới trang chủ tương ứng theo quyền (Dashboard cho Admin/Staff, Home cho Customer).

## 6. Business Rules (Quy Tắc Nghiệp Vụ)

- **BR-AUTH-01 (Định dạng Email):** Input `Email` phải đúng chuẩn định dạng email quốc tế (regex `^[a-zA-Z0-0_!#$%&'*+/=?`{|}~^.-]+@[a-zA-Z0-9.-]+$`).
- **BR-AUTH-02 (Bảo mật mật khẩu):** Mật khẩu gửi từ Client lên Server phải được truyền qua kết nối mã hóa TLS (HTTPS). Không được lưu trữ mật khẩu plain-text dưới database.
- **BR-AUTH-03 (Vòng đời Token):** Access Token có hiệu lực ngắn hạn (vd: 15-30 phút). Refresh Token có hiệu lực dài hạn (vd: 7-30 ngày).

## 7. Exception Handling (Xử Lý Ngoại Lệ)

Dưới đây là cách hệ thống phải bắt và xử lý các kịch bản lỗi, đặc biệt là các vấn đề bảo mật.

| Mã lỗi    | Tình huống ngoại lệ (Edge Case)                                                                      | Xử lý của hệ thống (System Response / Action)                                                                                                                                                                                                                                                                                                                                            | Thông báo lỗi trả về cho User                                                                                                                                                                                |
| :-------- | :--------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **EX-01** | **Người dùng nhập sai mật khẩu quá 5 lần.**                                                          | - Sai từ lần 1-4: Hệ thống tăng biến đếm `failed_attempts` lên 1 và từ chối đăng nhập.<br>- Lần thứ 5 sai: Hệ thống đổi trạng thái user thành `LOCKED`, khóa tài khoản trong khoảng thời gian nhất định (vd: 30 phút). Gửi Email cảnh báo bảo mật cho User.                                                                                                                              | - Lần 1-4: _"Email hoặc mật khẩu không chính xác."_<br>- Lần 5: _"Tài khoản của bạn đã bị khóa tạm thời do nhập sai mật khẩu quá 5 lần. Vui lòng thử lại sau 30 phút hoặc sử dụng tính năng Quên mật khẩu."_ |
| **EX-02** | **Token JWT hết hạn (Expired) trong khi phiên làm việc đang diễn ra.**                               | - Khi User gửi request bất kỳ lên API với Access Token đã hết hạn, hệ thống Filter bắt lỗi và trả về HTTP Status `401 Unauthorized` kèm mã code `TOKEN_EXPIRED`.<br>- Client sẽ dùng Refresh Token ngầm gọi API `/api/auth/refresh` để cấp đổi Access Token mới mà không làm gián đoạn trải nghiệm.<br>- Nếu Refresh Token cũng hết hạn, hệ thống buộc đăng xuất, yêu cầu đăng nhập lại. | (Người dùng thường không thấy nếu Refresh Token còn hạn). Nếu bắt buộc log-out: _"Phiên đăng nhập đã hết hạn. Vui lòng đăng nhập lại để tiếp tục."_                                                          |
| **EX-03** | **Tài khoản người dùng ở trạng thái Inactive (bị khóa/chưa kích hoạt) nhưng vẫn cố gắng đăng nhập.** | Dù thông tin Email và Password chính xác, hệ thống kiểm tra trường `status != ACTIVE` thì lập tức từ chối sinh Token. Bắn sự kiện Audit Log "Nỗ lực đăng nhập trên tài khoản vô hiệu hóa" và trả về HTTP Status `403 Forbidden`.                                                                                                                                                         | _"Tài khoản của bạn đang bị khóa hoặc chưa được kích hoạt. Vui lòng liên hệ Bộ phận CSKH để được hỗ trợ."_                                                                                                   |
| **EX-04** | Sai định dạng Email hoặc để trống các trường bắt buộc.                                               | Validation ở tầng Controller bắt lỗi ngay lập tức mà không cần chọc vào DB. Trả về HTTP `400 Bad Request`.                                                                                                                                                                                                                                                                               | _"Vui lòng nhập Email hợp lệ."_ hoặc _"Vui lòng không để trống mật khẩu."_                                                                                                                                   |
| **EX-05** | Email không tồn tại trong hệ thống.                                                                  | Trả về HTTP Status `401 Unauthorized`. (Lưu ý bảo mật: Dùng chung thông báo lỗi với sai mật khẩu để tránh kỹ thuật User Enumeration).                                                                                                                                                                                                                                                    | _"Email hoặc mật khẩu không chính xác."_                                                                                                                                                                     |

## 8. Postconditions (Điều Kiện Sau Đăng Nhập)

- Một chuỗi truy vết đăng nhập (Login Audit Trail) được ghi nhận trong cơ sở dữ liệu.
- Thời điểm đăng nhập cuối cùng (Last Login Time) của người dùng được cập nhật.
- Client nắm giữ Access Token để tiếp tục gửi vào Header `Authorization: Bearer <Token>` cho các request chức năng tiếp theo.

## 9. Acceptance Criteria (Tiêu Chí Nghiệm Thu)

- **AC1:** Người dùng đăng nhập thành công bằng email và mật khẩu đúng, nhận được JWT chứa đúng danh sách Role phân quyền.
- **AC2:** Hệ thống lập tức khóa tài khoản chính xác ở lần nhập sai mật khẩu thứ 5 và không cho phép đăng nhập kể cả lần thứ 6 có nhập đúng đi nữa.
- **AC3:** Nếu tài khoản bị set thành `Inactive` từ phía Admin, người dùng đó lập tức không thể đăng nhập kể từ thời điểm đó.
- **AC4:** Hệ thống không bao giờ trả về thông báo lỗi cho biết đích xác là "Email không tồn tại" (để chống dò quét email).
- **AC5:** Khi Access Token hết hạn, API trả về chuẩn xác HTTP Code 401 kèm body chứa keyword chỉ báo `ExpiredJwtException` để frontend tiến hành luồng refresh token.
