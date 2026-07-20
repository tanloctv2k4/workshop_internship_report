---
title : "Triển Khai Xác Thực Cognito"
date : 2026-07-14
weight : 4
chapter : false
pre : " <b> 5.3.2 </b> "
---

# Tài Liệu Hướng Dẫn Triển Khai Amazon Cognito User Pool - Hệ Thống PharmaCare

Tài liệu này hướng dẫn chi tiết quy trình thiết lập giải pháp xác thực người dùng (**Identity Management**) cho hệ thống **PharmaCare** thông qua **Amazon Cognito**. Nội dung bao gồm việc khởi tạo User Pool và phân quyền truy cập thông qua các nhóm người dùng (User Groups).

---

## 1. Khởi Tạo Amazon Cognito User Pool

### Các bước thực hiện:
1. Mở bảng điều khiển **Amazon Cognito Console**, chọn **Create user pool**.
2. Thiết lập cấu hình User Pool cho **PharmaCare**:
   * **User pool name:** `pharmacare-user-pool`.
   * Cấu hình các phương thức đăng nhập (Email/Password), chính sách mật khẩu (Password policy) và các tùy chọn bảo mật cần thiết.
3. Sau khi hoàn tất, hệ thống sẽ cung cấp các thông số định danh quan trọng:
   * **User Pool ID:** `ap-southeast-1_ccKkTEGWd`.
   * **Client ID:** `5pluo9vlml4kav6ib15vghsn2e`.

   ![Tổng quan cấu hình Cognito User Pool](/images/5-Workshop/5.3-lambda/cognito/cog1.png)

**Amazon Cognito User Pool** đóng vai trò là một "thư mục người dùng" (User Directory) bảo mật, cho phép **PharmaCare** quản lý thông tin đăng ký, đăng nhập và xác thực của người dùng mà không cần tự xây dựng cơ chế quản lý tài khoản phức tạp. Việc sử dụng Cognito giúp tách biệt logic xác thực khỏi mã nguồn ứng dụng, tăng cường khả năng mở rộng và đảm bảo các tiêu chuẩn bảo mật hiện đại.

---

## 2. Tạo Nhóm Người Dùng (User Groups)

### Các bước thực hiện:
1. Trong giao diện quản lý User Pool vừa tạo, tại mục **User management**, chọn **Groups**.
2. Click **Create group** để phân quyền người dùng:
   * **Admin Group:** Tạo nhóm `Admin` với mô tả "PharmaCare administrator group" (Precedence: 1).
   * **Customer Group:** Tạo nhóm `Customer` với mô tả "PharmaCare customer group" (Precedence: 2).

   ![Tạo nhóm Admin và Customer trong Cognito](/images/5-Workshop/5.3-lambda/cognito/cog2.png)

Việc phân chia nhóm người dùng (**User Groups**) cho phép **PharmaCare** thực hiện kiểm soát truy cập dựa trên vai trò (**Role-Based Access Control - RBAC**). 
* Nhóm **Admin** được cấp quyền quản trị các tài nguyên nhạy cảm và thao tác quản lý dữ liệu toàn hệ thống. 
* Nhóm **Customer** chỉ giới hạn quyền truy cập các chức năng dành cho người dùng cuối. 
Điều này đảm bảo rằng mỗi người dùng chỉ có thể thực hiện đúng các thao tác được phân quyền, giảm thiểu tối đa rủi ro từ việc lạm dụng hoặc truy cập trái phép.