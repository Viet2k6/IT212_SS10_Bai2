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

```text
# Functional Requirements – Authentication Module

## 1. Functional Overview

Module Authentication cho phép người dùng đăng nhập vào hệ thống Shop AI bằng email và mật khẩu. Sau khi xác thực thành công, hệ thống cấp JWT để người dùng truy cập các chức năng theo quyền được cấp thông qua RBAC.

---

## 2. Actors

- Admin
- Staff
- Customer

---

## 3. Preconditions

- Người dùng đã có tài khoản.
- Tài khoản tồn tại trong cơ sở dữ liệu.
- Hệ thống Authentication đang hoạt động.
- Người dùng có kết nối mạng.

---

## 4. Functional Requirements

### FR-01 Đăng nhập

- Hệ thống cho phép người dùng nhập email và mật khẩu.
- Hệ thống kiểm tra dữ liệu đầu vào.
- Hệ thống xác thực thông tin đăng nhập.
- Nếu hợp lệ, hệ thống sinh JWT.
- JWT chứa thông tin định danh và vai trò của người dùng.
- Hệ thống trả về Access Token cho client.

### FR-02 Phân quyền

- Sau khi đăng nhập thành công, hệ thống xác định vai trò người dùng.
- Người dùng chỉ được truy cập các chức năng được phép theo RBAC.
- Nếu truy cập trái quyền, hệ thống từ chối yêu cầu.

---

## 5. Main Flow

1. Người dùng nhập email và mật khẩu.
2. Hệ thống kiểm tra dữ liệu đầu vào.
3. Hệ thống tìm tài khoản theo email.
4. Kiểm tra trạng thái tài khoản.
5. Kiểm tra mật khẩu.
6. Xác định vai trò người dùng.
7. Sinh JWT.
8. Trả Access Token.
9. Người dùng sử dụng JWT để truy cập hệ thống.

---

## 6. Business Rules

- Email phải tồn tại.
- Mật khẩu phải chính xác.
- JWT phải còn hiệu lực.
- RBAC được áp dụng cho mọi yêu cầu.
- Chỉ tài khoản Active mới được đăng nhập.

---

## 7. Exception Handling

### EX-01 Sai mật khẩu quá 5 lần

- Hệ thống tăng bộ đếm số lần đăng nhập sai.
- Khi vượt quá 5 lần liên tiếp:
  - Khóa tạm thời tài khoản hoặc yêu cầu xác minh theo chính sách bảo mật.
  - Từ chối các lần đăng nhập tiếp theo.
  - Ghi log sự kiện bảo mật.
  - Thông báo người dùng tài khoản đã bị khóa tạm thời.

### EX-02 JWT hết hạn

- Khi JWT hết hạn trong phiên làm việc:
  - Hệ thống từ chối yêu cầu.
  - Trả về lỗi Unauthorized (401).
  - Yêu cầu người dùng đăng nhập lại để nhận JWT mới.
  - Không cho phép tiếp tục sử dụng token đã hết hạn.

### EX-03 Tài khoản Inactive

- Nếu tài khoản ở trạng thái Inactive:
  - Hệ thống từ chối đăng nhập.
  - Không sinh JWT.
  - Không tạo phiên làm việc.
  - Thông báo tài khoản đã bị khóa hoặc chưa được kích hoạt.

---

## 8. Postconditions

Nếu đăng nhập thành công:
- JWT được tạo.
- Người dùng được xác thực.
- Quyền truy cập được áp dụng theo RBAC.

Nếu đăng nhập thất bại:
- Không tạo JWT.
- Không tạo phiên làm việc.
- Hệ thống ghi nhận thông tin phục vụ giám sát bảo mật.

---

## 9. Acceptance Criteria

- Người dùng hợp lệ đăng nhập thành công và nhận JWT.
- Người dùng sai mật khẩu quá 5 lần bị xử lý theo chính sách bảo mật.
- JWT hết hạn không được phép tiếp tục sử dụng.
- Tài khoản Inactive không thể đăng nhập.
- RBAC được áp dụng chính xác cho mọi vai trò.
```
