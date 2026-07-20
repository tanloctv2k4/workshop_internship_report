---
title : "Triển Khai Lambda Migration DB"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.3.1 </b> "
---

## 1. Thiết Lập Quyền Thực Thi (IAM Role) Cho Lambda

### Các bước thực hiện:
1. Mở bảng điều khiển **IAM Console**, chọn mục **Roles** -> Click **Create role**.
2. Tạo IAM Role mới với định danh `pharmacare-lambda-role` và đính kèm 3 chính sách quyền (Permissions policies) phù hợp:
   * `AWSLambdaBasicExecutionRole` (AWS managed): Cho phép Lambda ghi log thực thi vào Amazon CloudWatch.
   * `AWSLambdaVPCAccessExecutionRole` (AWS managed): Cho phép Lambda khởi tạo card mạng riêng (ENI) để truy cập vào các tài nguyên nằm trong Private Subnet của VPC.
   * `pharmacare-read-rds-secret-policy` (Customer inline): Quyền tùy biến cho phép Lambda đọc mật khẩu kết nối database từ AWS Secrets Manager.

   ![Thiết lập IAM Role cho Lambda](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam1.png)

Việc áp dụng mô hình bảo mật **Đặc quyền tối thiểu (Least Privilege)** thông qua IAM Role giúp đảm bảo hàm Lambda chỉ sở hữu đúng các quyền hạn cần thiết để vận hành. Nhờ quyền VPC Access, hàm Lambda có thể giao tiếp an toàn với database RDS bên trong mạng kín mà không cần phơi bày giao thức kết nối ra Internet công cộng.

---

## 2. Khởi Tạo Hàm Lambda Migration

### Các bước thực hiện:
1. Truy cập bảng điều khiển **AWS Lambda Console**, chọn **Functions** -> Click **Create function**.
2. Chọn cấu hình **Author from scratch** và thiết lập các thông số cơ bản:
   * **Function name:** `pharmacare-db-migration`.
   * **Runtime:** Chọn **Node.js 22.x**.
   * **Architecture:** `x86_64`.
3. Tại phần **Permissions**, chọn **Use a custom role** và chỉ định IAM Role đã tạo ở Bước 1 (`pharmacare-lambda-role`).
4. Tại phần **Networking (VPC)**, kết nối Lambda vào hạ tầng mạng nội bộ:
   * **VPC:** Chọn `pharmacare-vpc`.
   * **Subnets:** Chọn 2 Private Subnets của hệ thống.
   * **Security groups:** Gắn Security Group chuyên biệt dành cho Lambda. Click **Create function** để khởi tạo.

   ![Cấu hình khởi tạo Lambda Migration](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam2.png)

Hàm Lambda `pharmacare-db-migration` đóng vai trò là công cụ tự động hóa khâu chuẩn bị dữ liệu nền tảng cho hệ thống **PharmaCare**. Thay vì cấu hình thủ công hoặc kết nối trực tiếp từ máy cá nhân vào RDS (gây mất an toàn), Lambda khởi chạy tự động bên trong mạng nội bộ VPC, đảm bảo tính nhất quán và bảo mật tuyệt đối cho quy trình triển khai.

---

## 3. Điều Chỉnh Tài Nguyên & Thời Gian Chờ (Timeout)

### Các bước thực hiện:
1. Tại giao diện quản lý hàm Lambda `pharmacare-db-migration`, chọn tab **Configuration** -> Chọn mục **General configuration** -> Click **Edit**.
2. Điều chỉnh cấu hình tài nguyên thực thi:
   * **Memory:** Tăng lên `256 MB` (đảm bảo đủ bộ nhớ để tải thư viện và xử lý kết nối DB).
   * **Timeout:** Thiết lập thành `1 min 0 sec` (60 giây). Click **Save** để lưu lại cấu hình.

   ![Cấu hình Timeout và Memory cho Lambda](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam3.png)

   ![Xác nhận lưu cấu hình General settings](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam4.png)

Mặc định, AWS Lambda có thời gian chờ (timeout) chỉ là 3 giây. Quy trình migration đòi hỏi kết nối tới RDS, đọc mật khẩu từ Secrets Manager và thực thi các câu lệnh SQL khởi tạo cấu trúc nhiều bảng phức tạp của **PharmaCare** (như `users`, toa thuốc, danh mục sản phẩm). Việc tăng timeout lên 1 phút giúp ngắt hoàn toàn tình trạng lỗi ngắt kết nối giữa chừng (Timeout Exception) trong các đợt cập nhật lớn của hệ thống.

---

## 4. Thiết Lập Biến Môi Trường (Environment Variables)

### Các bước thực hiện:
1. Tại tab **Configuration**, chọn mục **Environment variables** -> Click **Edit**.
2. Lần lượt thêm các cặp Key-Value định danh thông số kết nối cơ sở dữ liệu:
   * `DB_HOST`: Endpoint của máy chủ RDS (ví dụ: `pharmacare-db.cpiq4y2wkyzx.ap-southeast-1.rds.amazonaws.com`).
   * `DB_NAME`: Tên cơ sở dữ liệu (`pharmacare_ai`).
   * `DB_PORT`: Cổng truy cập (`5432`).
   * `RDS_SECRET_ARN`: Đường dẫn định danh ARN của secret lưu mật khẩu trong AWS Secrets Manager.
3. Click **Save** để áp dụng.

   ![Thiết lập biến môi trường cho Lambda](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam5.png)

Biến môi trường giúp tách biệt hoàn toàn thông tin cấu hình hạ tầng khỏi mã nguồn ứng dụng. Nhờ cơ chế này, bạn có thể tái sử dụng hoặc linh hoạt thay đổi máy chủ cơ sở dữ liệu (từ môi trường Test/Dev sang Production) mà không cần chỉnh sửa hoặc viết chết (hardcode) các thông số trong code, giữ cho hệ thống luôn tuân thủ nguyên tắc bảo mật và quản trị mã nguồn sạch.

---

## 5. Xây Dựng, Đóng Gói Và Cập Nhật Mã Nguồn Triển Khai

### Các bước thực hiện:
1. Tại môi trường phát triển cục bộ (VS Code), khởi tạo dự án Node.js (`npm init -y`) và cài đặt các thư viện phụ thuộc cần thiết để kết nối RDS và đọc Secrets Manager (`pg`, `@aws-sdk/client-secrets-manager`). Viết mã nguồn migration trong file `index.mjs` chứa các câu lệnh SQL tạo bảng.
2. Sử dụng lệnh PowerShell `Compress-Archive` trong Terminal để đóng gói mã nguồn (`index.mjs`), thư mục `node_modules` và cấu hình thành tệp nén `function.zip` lần thứ nhất.

   ![Khởi tạo và nén mã nguồn lần 1 trong VS Code](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam6.png)

3. Quay lại AWS Lambda Console, tại tab **Code**, click **Upload from** -> **.zip file**, chọn tệp `function.zip` vừa nén để tiến hành tải lên.

   ![Giao diện tải tệp nén ZIP cập nhật Lambda lần 1](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam7.png)

4. Sau khi cập nhật, kiểm tra lại mã nguồn hiển thị trực tiếp trên trình biên soạn (Code source) của Lambda Console nhằm đảm bảo cấu trúc lệnh SQL migration đã khớp hoàn toàn.

   ![Kiểm tra mã nguồn Lambda sau khi update lần 1](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam8.png)

5. Trong trường hợp cần chỉnh sửa, bổ sung thêm bảng mới hoặc cập nhật logic, tiến hành nén lại tệp `function.zip` lần cuối trên VS Code và lặp lại thao tác tải lên giao diện **Upload from .zip file**.

   ![Giao diện tải tệp nén ZIP cập nhật Lambda lần cuối](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam9.png)

6. Xác nhận cập nhật thành công (thông báo `Successfully updated the function`), mã nguồn mới nhất sẵn sàng để kích hoạt chạy thực tế vào RDS PostgreSQL.

   ![Mã nguồn Migration hoàn chỉnh sau khi cập nhật thành công](/workshop_internship_report/images/5-Workshop/5.3-lambda/lambda/lam10.png)

Quy trình đóng gói bằng tệp `.zip` đảm bảo toàn bộ thư viện phụ thuộc của Node.js (`node_modules`) được tải trực tiếp vào môi trường thực thi của Lambda. Điều này giúp hàm Lambda có đầy đủ công cụ để tự động lấy mật khẩu giải mã từ Secrets Manager và thực thi các khối lệnh SQL, khởi tạo cấu trúc bảng dữ liệu hoàn chỉnh, chuẩn bị nền tảng sẵn sàng cho toàn bộ kiến trúc microservices và trí tuệ nhân tạo của **PharmaCare** hoạt động.