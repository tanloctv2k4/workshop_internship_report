---
title : "Triển Khai Cơ Sở Dữ Liệu"
date : 2026-07-14
weight : 2
chapter : false
pre : " <b> 5.2.2 </b> "
---

## 1. Khởi Tạo Nhóm Subnet Cho Cơ Sở Dữ Liệu (DB Subnet Group)

### Các bước thực hiện:
1. Mở bảng điều khiển [Amazon RDS Console](https://ap-southeast-1.console.aws.amazon.com/rds/home?region=ap-southeast-1#). Tại thanh điều hướng bên trái, chọn **Subnet groups** -> Click **Create DB subnet group**.
2. Cấu hình các thông số chi tiết:
   * **Name:** `pharmacare-db-subnet-group`.
   * **Description:** `DB subnet group for PharmaCare AI RDS PostgreSQL`.
   * **VPC:** Chọn VPC của hệ thống (`pharmacare-vpc` | `vpc-00b5763254ecfdf41`).
3. Trong phần **Add subnets**, chọn 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`) và chỉ định đúng 2 Private DB Subnets (`10.0.11.0/24`, `10.0.12.0/24`). Nhấn **Create** để hoàn tất.

   ![Cấu hình DB Subnet Group](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data1.png)

**DB Subnet Group** là tập hợp các mạng con (subnets) mà bạn chỉ định cho máy chủ cơ sở dữ liệu Amazon RDS trong một VPC. Việc lựa chọn subnets thuộc 2 vùng sẵn sàng (Multi-AZ) khác nhau đảm bảo tính sẵn sàng cao và khả năng khôi phục sau thảm họa (Disaster Recovery). Đồng thời, việc đặt database hoàn toàn bên trong lớp **Private DB Subnets** giúp cô lập tầng dữ liệu khỏi internet công cộng, ngăn chặn tuyệt đối các truy cập trái phép từ bên ngoài.

---

## 2. Khởi Tạo Máy Chủ Cơ Sở Dữ Liệu Amazon RDS PostgreSQL

### Các bước thực hiện:
1. Tại RDS Console, chọn mục **Databases** -> Click **Create database**.
2. Chọn phương thức tạo **Standard create** và cấu hình máy chủ:
   * **Engine options:** Chọn **PostgreSQL**.
   * **DB instance identifier:** `pharmacare-db`.
   * **Instance configuration:** Chọn lớp máy chủ phù hợp với quy mô hệ thống (ví dụ: `db.t4g.micro`).
   * **Connectivity:** Gán vào VPC `pharmacare-vpc`, chọn DB Subnet group `pharmacare-db-subnet-group`, và gắn Security Group bảo vệ dành riêng cho RDS (`pharmacare-rds-sg`).
3. Kiểm tra lại cấu hình và click **Create database**. Chờ đến khi trạng thái máy chủ chuyển sang `Available`.

   ![Khởi tạo thành công Amazon RDS PostgreSQL](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data2.png)

**Amazon RDS for PostgreSQL** cung cấp một cơ sở dữ liệu quan hệ được quản lý hoàn toàn (Fully Managed Database), chịu trách nhiệm xử lý toàn bộ dữ liệu nghiệp vụ cốt lõi của **PharmaCare** (như thông tin người dùng, danh mục thuốc, hồ sơ bệnh án, lịch sử tư vấn). Việc sử dụng dịch vụ managed giúp tự động hóa các tác vụ quản trị nặng nhọc như sao lưu dữ liệu (Automated Backups), cập nhật bản vá lỗi phần mềm (Patching), và theo dõi hiệu suất, giúp đội ngũ tập trung hoàn toàn vào việc phát triển logic ứng dụng.

---

## 3. Quản Lý Thông Tin Đăng Nhập Với AWS Secrets Manager

### Các bước thực hiện:
1. Trong quá trình khởi tạo RDS (hoặc quản lý credentials sau khi tạo), hệ thống tự động tích hợp với **AWS Secrets Manager** để tạo một secret bảo mật lưu trữ mật khẩu master của database.
2. Mở **AWS Secrets Manager Console**, chọn **Secrets** để kiểm tra secret vừa được tạo (ví dụ: `rds!db-b1798063-e2c6-417e-9293-b98b072f7341`).

   ![AWS Secrets Manager lưu trữ mật khẩu RDS](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data3.png)

**AWS Secrets Manager** giải quyết bài toán bảo mật thông tin nhạy cảm. Thay vì ghi chết (hardcode) tài khoản và mật khẩu database trong mã nguồn hoặc biến môi trường — nơi rất dễ bị rò rỉ — Secrets Manager sẽ mã hóa và lưu trữ thông tin này. Khi các dịch vụ backend (như Lambda Functions hoặc App Servers) cần kết nối vào database, chúng sẽ gọi API tới Secrets Manager để lấy mật khẩu một cách an toàn và động theo thời gian thực.

---

## 4. Cấu Hình Security Group Cho VPC Endpoint

### Các bước thực hiện:
1. Truy cập **VPC Console** -> Chọn **Security Groups** -> Click **Create security group**.
2. Cấu hình nhóm bảo mật dành riêng cho Endpoint kết nối Secrets Manager:
   * **Security group name:** `pharmacare-vpce-sg`.
   * **Description:** `Security group for PharmaCare VPC Endpoints`.
   * **VPC:** Chọn `pharmacare-vpc`.
3. Tại phần **Inbound rules**, thêm quy tắc cho phép giao thức **HTTPS (Port 443)** từ nguồn là Security Group của các ứng dụng backend (ví dụ: Security Group của Lambda `sg-03dd8ac64cbc76d6d`). Click **Create security group**.

   ![Cấu hình Security Group cho VPC Endpoint](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data4.png)

Security group `pharmacare-vpce-sg` đóng vai trò là chốt chặn bảo mật ở tầng mạng cho đường dẫn nội bộ tới Secrets Manager. Nó áp dụng nguyên tắc đặc quyền tối thiểu, chỉ cho phép luồng dữ liệu được mã hóa qua giao thức HTTPS (Cổng 443) đi vào từ đúng các tài nguyên nội bộ được cấp phép (như máy chủ Backend hoặc Lambda), ngăn chặn rà quét trái phép từ các phân vùng mạng khác.

---

## 5. Triển Khai VPC Endpoint Cho Secrets Manager

### Các bước thực hiện:
1. Tại **VPC Console**, chọn mục **Endpoints** -> Click **Create endpoint**.
2. Cấu hình thông số Endpoint:
   * **Name tag:** `pharmacare-secretsmanager-vpce`.
   * **Service category:** Chọn **AWS services**.
   * **Service name:** Tìm và chọn `com.amazonaws.ap-southeast-1.secretsmanager` (Loại: **Interface**).
   * **VPC:** Chọn `pharmacare-vpc`.
   * **Subnets:** Chọn các subnets riêng (Private Subnets) nơi ứng dụng của bạn vận hành.
   * **Security groups:** Gắn Security Group vừa tạo ở bước 4 (`pharmacare-vpce-sg`).
3. Nhấn **Create endpoint** và đợi đến khi trạng thái hiển thị là `Available`.

   ![Trạng thái VPC Endpoint cho Secrets Manager](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data5.png)

Đây là bước tối ưu hóa bảo mật kiến trúc mang tính quyết định. Mặc định, Secrets Manager là dịch vụ công cộng của AWS; nếu không có Endpoint, ứng dụng trong Private Subnet sẽ phải đi qua NAT Gateway và vòng ra Internet công cộng để lấy mật khẩu DB. Việc khởi tạo **Interface VPC Endpoint (sử dụng AWS PrivateLink)** tạo ra một đường ống kết nối riêng tư trực tiếp từ mạng nội bộ VPC thẳng tới Secrets Manager thông qua địa chỉ IP nội bộ (Private DNS). Điều này giúp:
* **Tối đa hóa bảo mật:** Thông tin mật khẩu database không bao giờ rời khỏi mạng riêng ảo AWS.
* **Tăng tốc độ & Giảm chi phí:** Giảm độ trễ kết nối (latency) và tiết kiệm chi phí xử lý dữ liệu qua NAT Gateway cho hệ thống PharmaCare.