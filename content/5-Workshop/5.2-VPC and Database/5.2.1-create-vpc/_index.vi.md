---
title : "Tạo VPC"
date : 2026-07-14
weight : 1
chapter : false
pre : " <b> 5.2.1 </b> "
---

## 1. Khởi Tạo Virtual Private Cloud (VPC)

### Các bước thực hiện:
1. Mở [Amazon VPC console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1#Home).
2. Trong thanh điều hướng, chọn **Your VPCs**, click **Create VPC**.
   
   ![Cấu hình tạo VPC](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc1.png)
   
3. Cấu hình các thông số chi tiết theo mô hình **VPC and more**:
   * **Name tag auto-generation:** `pharmacare-vpc` (hoặc định danh hệ thống tương đương).
   * **IPv4 CIDR block:** `10.0.0.0/16` (Cung cấp tối đa khoảng 65,536 địa chỉ IP).
   * **Number of Availability Zones (AZs):** Chọn `2` (Tăng cường tính sẵn sàng cao).
   * **Number of public subnets:** `2`.
   * **Number of private subnets:** `4` (Chia cho tầng App và tầng DB).
4. Xác nhận lại sơ đồ trực quan và nhấn nút **Create VPC** để hệ thống tự động khởi tạo.

   ![Xác nhận tạo VPC thành công](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc2.png)

Bước này thiết lập một môi trường mạng riêng ảo (VPC) cô lập và hoàn toàn độc lập trên hạ tầng đám mây AWS thuộc khu vực Singapore (`ap-southeast-1`). Sử dụng tính năng "VPC and more" giúp tự động hóa quá trình liên kết các thành phần cốt lõi, tạo nên một khung xương vững chắc, bảo mật và chuẩn hóa ngay từ ban đầu để triển khai các dịch vụ của PharmaCare.

--- 

## 2. Kiểm Tra Và Cấu Hình Các Phân Mạng (Subnets)

### Các bước thực hiện:
1. Trong thanh điều hướng bên trái, chọn mục **Subnets**.
2. Kiểm tra danh sách **6 Subnets** đã được tạo tự động và đảm bảo trạng thái hiển thị là `Available`.

   ![Danh sách Subnets trong hệ thống](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc3.png)

Việc chia nhỏ không gian mạng lớn (`10.0.0.0/16`) thành **6 Subnets** nhỏ hơn (`/24`) và phân bố đều trên 2 vùng sẵn sàng (Availability Zones: `ap-southeast-1a` và `ap-southeast-1b`) giúp hệ thống PharmaCare có khả năng chống chịu lỗi (Fault Tolerance). Nếu một trung tâm dữ liệu vật lý của AWS gặp sự cố, hệ thống vẫn hoạt động bình thường trên vùng còn lại. Đồng thời, việc phân tách subnets giúp cô lập bảo mật theo các lớp kiến trúc rõ rệt:
* **Public Subnets (`10.0.1.0/24`, `10.0.2.0/24`):** Nơi tiếp nhận các kết nối công khai từ Internet (ví dụ: Load Balancer, Bastion Host).
* **Private App Subnets (`10.0.21.0/24`, `10.0.22.0/24`):** Đảm nhận xử lý logic core backend, API ứng dụng. Lớp này bị ẩn hoàn toàn với môi trường Internet.
* **Private DB Subnets (`10.0.11.0/24`, `10.0.12.0/24`):** Lớp mạng sâu nhất, dùng bảo vệ cơ sở dữ liệu (RDS, OpenSearch, Cache), ngăn chặn tối đa nguy cơ rò rỉ dữ liệu.

---

## 3. Kiểm Tra Và Liên Kết Bảng Định Tuyến (Route Tables)

### Các bước thực hiện:
1. Trong thanh điều hướng, chọn mục **Route tables**.
2. Kiểm tra bảng định tuyến mạng kín `pharmacare-private-rt` và đảm bảo nó đã liên kết thành công (Explicit subnet associations) với **4 Private Subnets**.

   ![Cấu hình Route Table Private](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc4.png)

3. Tiếp tục kiểm tra bảng định tuyến mạng công cộng `pharmacare-public-rt` và đảm bảo nó đã liên kết với **2 Public Subnets**.

   ![Cấu hình Route Table Public](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc5.png)

Bảng định tuyến (Route Tables) đóng vai trò như hệ thống điều hướng giao thông điều khiển luồng dữ liệu (traffic) di chuyển ra vào các vùng mạng:
* **Bảng Public Route:** Cho phép lưu lượng mạng từ các Public Subnets đi thẳng tới Internet Gateway để tương tác trực tiếp với khách hàng hoặc người dùng cuối.
* **Bảng Private Route:** Chỉ cho phép các dữ liệu giao tiếp nội bộ trong mạng (Local route) hoặc kết nối ra ngoài một chiều khi cần thiết. Lớp cấu hình này khóa chặt luồng truy cập trái phép hướng từ ngoài Internet vào thẳng máy chủ ứng dụng hoặc DB của PharmaCare.

---

## 4. Kiểm Tra Cổng Internet (Internet Gateway - IGW)

### Các bước thực hiện:
1. Trong thanh điều hướng, chọn **Internet gateways**.
2. Xác nhận cổng Internet có tên `pharmacare-igw` đã được khởi tạo và đang ở trạng thái `Attached` (Đã gắn kết) vào đúng VPC ID của mạng hệ thống.

   ![Trạng thái Internet Gateway](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc6.png)

Internet Gateway (`pharmacare-igw`) chính là "cửa ngõ" trung gian duy nhất kết nối mạng VPC nội bộ của PharmaCare với thế giới Internet bên ngoài. Thiếu IGW, hệ thống mạng sẽ hoàn toàn biệt lập. Thành phần này hỗ trợ định tuyến để các dịch vụ công khai (như giao diện web hoặc cổng API Gateway) có thể đón nhận các yêu cầu truy cập từ khách hàng, cũng như cho phép hệ thống tải và cập nhật các gói thư viện phần mềm cần thiết.

---

## 5. Cấu Hình Các Nhóm Bảo Mật (Security Groups)

### Các bước thực hiện:
1. Trong thanh điều hướng bên trái, di chuyển xuống mục **Security** và chọn **Security Groups**.
2. Tiến hành kiểm tra hoặc click **Create security group** để khởi tạo tường lửa cho từng nhóm dịch vụ riêng biệt.

   ![Danh sách Security Groups](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/vpc/vpc7.png)

Security Groups đóng vai trò như các bức tường lửa ảo (Virtual Firewall) hoạt động ở cấp độ phần mềm/máy chủ trực tiếp. Việc phân tách rõ ràng các nhóm như `pharmacare-lambda-sg`, `pharmacare-opensearch-sg`, và `pharmacare-rds-sg` giúp triển khai quy tắc **"đặc quyền tối thiểu" (Least Privilege)**:
* Cho phép kiểm soát chặt chẽ chính xác cổng (Port) và địa chỉ IP nguồn nào được quyền gửi dữ liệu đến (Inbound) hoặc phản hồi đi (Outbound).
* **Ví dụ:** Chỉ cho phép các máy chủ backend thuộc nhóm App Subnet mới có quyền tạo phiên kết nối tới Cổng cơ sở dữ liệu (`pharmacare-rds-sg`), chặn đứng các hành vi rà quét port hoặc tấn công khai thác lỗ hổng mạng nội bộ từ các tài nguyên không được phân quyền.