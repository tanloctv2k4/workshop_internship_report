---
title : "Triển Khai Website PharmaCare Lên Amazon S3 Và CloudFront"
date : 2026-07-19
weight : 8
chapter : false
pre : " <b> 5.6 </b> "
---

## 1. Chuẩn Bị Bản Build Frontend

Trước khi triển khai website lên AWS, mã nguồn React cần được biên dịch thành các tệp tĩnh tối ưu cho môi trường production. Với dự án sử dụng Vite, kết quả build được tạo trong thư mục `dist`.

### Các bước thực hiện:

1. Mở dự án frontend PharmaCare bằng Visual Studio Code.
2. Kiểm tra các biến môi trường production, đặc biệt là endpoint của API Gateway, API chatbot và thông tin Amazon Cognito.
3. Mở Terminal tại thư mục dự án và cài đặt các thư viện cần thiết:

   ```bash
   npm install
   ```

4. Thực hiện lệnh build:

   ```bash
   npm run build
   ```

5. Sau khi lệnh hoàn tất, kiểm tra thư mục:

   ```text
   dist/
   ```

6. Thư mục `dist` thông thường bao gồm:

   ```text
   dist/
   ├── assets/
   ├── favicon.svg
   ├── index.html
   └── vite.svg
   ```

Toàn bộ nội dung trong thư mục `dist` là các tài nguyên tĩnh sẽ được tải lên Amazon S3. Không tải toàn bộ mã nguồn dự án, thư mục `src` hoặc `node_modules` lên bucket frontend.

---

## 2. Tạo Amazon S3 Bucket Lưu Trữ Frontend

Amazon S3 được sử dụng làm origin lưu trữ các tệp HTML, CSS, JavaScript và tài nguyên tĩnh của website PharmaCare.

### Các bước thực hiện:

1. Mở [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1).
2. Trong thanh điều hướng bên trái, chọn **Buckets**.
3. Nhấn **Create bucket**.
4. Cấu hình bucket:

   * **AWS Region:** `Asia Pacific (Singapore) ap-southeast-1`.
   * **Bucket type:** Chọn **General purpose**.
   * **Bucket namespace:** Chọn **Global namespace**.
   * **Bucket name:** `pharmacare-frontend-web-phu-2026`.
   * **Object Ownership:** Chọn **ACLs disabled (recommended)**.
   * **Block Public Access:** Bật **Block all public access**.
   * **Bucket Versioning:** Có thể để **Disable** trong môi trường phát triển.
   * **Default encryption:** Chọn mã hóa phía máy chủ **SSE-S3**.

5. Kiểm tra lại cấu hình và nhấn **Create bucket**.

   ![Tạo S3 bucket lưu trữ frontend PharmaCare](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy1.jpg)

Sau khi tạo thành công, mở bucket và xác nhận giao diện **Objects** chưa có dữ liệu.

   ![S3 bucket frontend được tạo thành công](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy2.jpg)

Bucket được giữ ở trạng thái private. Người dùng không truy cập trực tiếp vào S3 mà truy cập website thông qua Amazon CloudFront. Cách triển khai này giúp che giấu origin, tăng cường bảo mật và tránh mở public bucket không cần thiết.

---

## 3. Tải Nội Dung Thư Mục `dist` Lên Amazon S3

### Các bước thực hiện:

1. Mở bucket:

   ```text
   pharmacare-frontend-web-phu-2026
   ```

2. Tại tab **Objects**, nhấn **Upload**.
3. Chọn **Add files** và **Add folder**.
4. Chọn toàn bộ nội dung bên trong thư mục `dist`.

> Không tải thư mục `dist` thành một thư mục con. Tệp `index.html` phải nằm trực tiếp ở thư mục gốc của bucket.

5. Giữ Storage Class ở mức **S3 Standard**.
6. Nhấn **Upload** và chờ quá trình hoàn tất.
7. Kiểm tra bucket có các đối tượng chính:

   ```text
   assets/
   favicon.svg
   index.html
   vite.svg
   ```

   ![Tải các tệp build trong thư mục dist lên Amazon S3](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy3.jpg)

Có thể triển khai bằng AWS CLI để đồng bộ nhanh hơn:

```bash
aws s3 sync dist/ s3://pharmacare-frontend-web-phu-2026 --delete
```

Tùy chọn `--delete` xóa các tệp cũ trên S3 không còn tồn tại trong bản build mới. Vì vậy, cần kiểm tra đúng tên bucket trước khi chạy lệnh.

---

## 4. Tạo Amazon CloudFront Distribution Cho Website

Amazon CloudFront được sử dụng để phân phối website đến người dùng thông qua mạng lưới máy chủ biên toàn cầu. CloudFront lấy nội dung từ S3, lưu vào bộ nhớ đệm và phục vụ qua HTTPS.

### Các bước thực hiện:

1. Mở [Amazon CloudFront Console](https://console.aws.amazon.com/cloudfront/v3/home).
2. Chọn **Distributions** và nhấn **Create distribution**.
3. Cấu hình thông tin cơ bản:

   * **Distribution name:** `pharmacare-frontend-distribution`.
   * **Description:** `CloudFront distribution for PharmaCare AI React frontend`.
   * **Distribution type:** Chọn website hoặc ứng dụng đơn.
   * **Origin type:** Chọn **Amazon S3**.
   * **Origin:** Chọn bucket `pharmacare-frontend-web-phu-2026`.
   * **Private S3 access:** Cho phép CloudFront truy cập bucket private bằng cơ chế được AWS khuyến nghị.
   * **Viewer protocol policy:** Chọn chuyển hướng HTTP sang HTTPS.
   * **Allowed HTTP methods:** Chọn `GET` và `HEAD`.
   * **Cache policy:** Sử dụng chính sách tối ưu cho nội dung tĩnh.
   * **Compress objects automatically:** Bật.
   * **Default root object:** Nhập `index.html`.

4. Kiểm tra toàn bộ cấu hình và nhấn **Create distribution**.
5. Chờ trạng thái distribution chuyển sang **Enabled**.

   ![Tạo CloudFront Distribution cho website PharmaCare](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy4.jpg)

Sau khi triển khai thành công, CloudFront cung cấp một domain có dạng:

```text
https://xxxxxxxxxxxxxx.cloudfront.net
```

Domain này là địa chỉ public tạm thời để kiểm tra website trước khi cấu hình domain riêng.

### Vai trò của CloudFront

* Phân phối nội dung từ Edge Location gần người dùng nhất.
* Cung cấp HTTPS mặc định cho website.
* Hạn chế truy cập trực tiếp vào S3 origin.
* Lưu cache tệp HTML, CSS, JavaScript và hình ảnh.
* Giảm độ trễ và số lượng request trực tiếp đến Amazon S3.
* Hỗ trợ tích hợp AWS WAF và domain tùy chỉnh.

---

## 5. Cấu Hình Error Pages Cho React Single Page Application

React sử dụng cơ chế định tuyến phía client. Khi người dùng truy cập trực tiếp một đường dẫn như `/products`, `/cart` hoặc `/orders`, S3 có thể không tìm thấy tệp tương ứng và trả về lỗi `403` hoặc `404`.

Để CloudFront luôn trả về `index.html` cho React Router xử lý, cần tạo Custom Error Response.

### Các bước thực hiện:

1. Mở CloudFront Distribution `pharmacare-frontend-distribution`.
2. Chọn tab **Error pages**.
3. Nhấn **Create custom error response**.
4. Tạo cấu hình cho lỗi `403`:

   * **HTTP error code:** `403`.
   * **Customize error response:** `Yes`.
   * **Response page path:** `/index.html`.
   * **HTTP response code:** `200`.
   * **Error caching minimum TTL:** Có thể đặt `0` trong quá trình phát triển.

5. Tạo thêm cấu hình cho lỗi `404`:

   * **HTTP error code:** `404`.
   * **Response page path:** `/index.html`.
   * **HTTP response code:** `200`.

   ![Cấu hình CloudFront Error Pages cho React Router](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy5.jpg)

Sau khi cấu hình, các URL sau vẫn được CloudFront trả về ứng dụng React:

```text
/products
/products/PC-0001
/cart
/checkout
/orders
/admin/products
```

React Router tiếp tục xác định component cần hiển thị ở phía trình duyệt.

---

## 6. Xóa Cache CloudFront Bằng Invalidation

CloudFront có thể tiếp tục phục vụ bản build cũ do nội dung đã được lưu tại Edge Location. Sau mỗi lần cập nhật frontend, cần tạo invalidation để CloudFront lấy nội dung mới từ S3.

### Các bước thực hiện:

1. Mở CloudFront Distribution.
2. Chọn tab **Invalidations**.
3. Nhấn **Create invalidation**.
4. Nhập đường dẫn:

   ```text
   /*
   ```

5. Nhấn **Create invalidation**.
6. Chờ trạng thái chuyển từ `In progress` sang `Completed`.

   ![Tạo CloudFront Invalidation sau khi cập nhật frontend](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy6.png)

Có thể thực hiện bằng AWS CLI:

```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

Quy trình cập nhật website sau này được rút gọn thành:

```text
Chỉnh sửa mã nguồn React
        ↓
npm run build
        ↓
Đồng bộ dist lên S3
        ↓
Tạo CloudFront Invalidation
        ↓
Kiểm tra website
```

---

## 7. Cập Nhật Callback URL Trong Amazon Cognito

Sau khi frontend có CloudFront domain, cần cập nhật Amazon Cognito để quá trình đăng nhập và đăng xuất chuyển hướng đúng về website production.

### Các bước thực hiện:

1. Mở [Amazon Cognito Console](https://ap-southeast-1.console.aws.amazon.com/cognito/v2/idp/user-pools?region=ap-southeast-1).
2. Chọn User Pool của hệ thống PharmaCare.
3. Chọn **App clients** và mở App Client của frontend.
4. Chọn mục cấu hình **Login pages**.
5. Thêm CloudFront domain vào **Allowed callback URLs**, ví dụ:

   ```text
   https://xxxxxxxxxxxxxx.cloudfront.net/
   ```

6. Thêm domain tương tự vào **Allowed sign-out URLs**:

   ```text
   https://xxxxxxxxxxxxxx.cloudfront.net/
   ```

7. Có thể tiếp tục giữ địa chỉ localhost phục vụ phát triển:

   ```text
   http://localhost:5173/
   ```

8. Nhấn **Save changes**.

   ![Cập nhật Callback URL và Sign-out URL trong Amazon Cognito](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy7.png)

Khi sử dụng domain riêng, cần bổ sung domain đó vào cả callback URL và sign-out URL. URL phải khớp chính xác giao thức, tên miền, port và đường dẫn; nếu không, Cognito có thể trả về lỗi `redirect_mismatch`.

---

## 8. Đăng Ký Domain Bằng Amazon Route 53

Amazon Route 53 được sử dụng để tìm kiếm, đăng ký và quản lý domain cho website PharmaCare.

### Các bước thực hiện:

1. Mở [Amazon Route 53 Console](https://console.aws.amazon.com/route53/).
2. Chọn **Registered domains**.
3. Nhấn **Register domains**.
4. Nhập tên domain mong muốn vào ô tìm kiếm.
5. Kiểm tra trạng thái khả dụng và chi phí đăng ký.
6. Nếu domain mong muốn không khả dụng, chọn một tên thay thế phù hợp.
7. Thêm domain vào danh sách lựa chọn.
8. Nhập thông tin chủ sở hữu domain.
9. Bật hoặc kiểm tra tính năng bảo vệ thông tin đăng ký nếu được hỗ trợ.
10. Hoàn tất thanh toán và xác minh email đăng ký.

   ![Tìm kiếm và đăng ký domain cho PharmaCare trong Route 53](/workshop_internship_report/images/5-Workshop/5.6-deploy-web/deploy8.jpg)

Ảnh kiểm tra cho thấy domain được tìm kiếm có thể không còn khả dụng. Trong trường hợp này, cần chọn một domain thay thế trong danh sách gợi ý hoặc sử dụng một tên miền khác phù hợp với thương hiệu PharmaCare.

> Việc tìm kiếm domain không đồng nghĩa với đã đăng ký thành công. Domain chỉ thuộc quyền quản lý của tài khoản sau khi hoàn tất thanh toán, xác minh và Route 53 hiển thị trạng thái đăng ký thành công.

---

## 9. Gắn Domain Riêng Vào CloudFront

Sau khi đăng ký domain, cần tạo chứng chỉ SSL/TLS và liên kết domain với CloudFront.

### 9.1. Yêu Cầu Chứng Chỉ Trong AWS Certificate Manager

1. Chuyển AWS Region sang:

   ```text
   US East (N. Virginia) us-east-1
   ```

2. Mở AWS Certificate Manager.
3. Nhấn **Request certificate**.
4. Chọn **Request a public certificate**.
5. Nhập domain, ví dụ:

   ```text
   pharmacare.example
   www.pharmacare.example
   ```

6. Chọn phương thức xác minh DNS.
7. Tạo các bản ghi xác minh trong Route 53.
8. Chờ trạng thái chứng chỉ chuyển sang **Issued**.

Chứng chỉ dùng cho CloudFront phải được tạo tại Region `us-east-1`.

### 9.2. Cập Nhật CloudFront Distribution

1. Mở Distribution `pharmacare-frontend-distribution`.
2. Chọn **Settings** và nhấn **Edit**.
3. Thêm domain vào **Alternate domain names (CNAMEs)**.
4. Chọn chứng chỉ ACM đã được cấp.
5. Lưu thay đổi và chờ CloudFront triển khai lại.

### 9.3. Tạo DNS Record Trong Route 53

1. Mở Hosted Zone của domain.
2. Nhấn **Create record**.
3. Tạo bản ghi:

   * **Record type:** `A`.
   * **Alias:** Bật.
   * **Route traffic to:** Alias to CloudFront distribution.
   * **Distribution:** Chọn `pharmacare-frontend-distribution`.

4. Nếu sử dụng `www`, tạo thêm bản ghi tương ứng.
5. Chờ DNS được cập nhật và truy cập website bằng domain mới.

---
## 10. Quy Trình Triển Khai Phiên Bản Mới

Khi cần cập nhật frontend, thực hiện:

```bash
npm run build
aws s3 sync dist/ s3://pharmacare-frontend-web-phu-2026 --delete
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

Quy trình tổng thể:

```text
React Source Code
        ↓
Vite Production Build
        ↓
dist/
        ↓
Amazon S3 Private Bucket
        ↓
Amazon CloudFront
        ↓
Route 53 Custom Domain
        ↓
Người dùng truy cập bằng HTTPS
```

Amazon Cognito, API Gateway và Lambda tiếp tục xử lý xác thực và nghiệp vụ backend. CloudFront chỉ chịu trách nhiệm phân phối frontend tĩnh đến người dùng.

---

## 11. Kết Quả Đạt Được

Sau khi hoàn thành quá trình triển khai:

* Frontend React được build thành các tệp tĩnh production.
* Các tệp được lưu trữ trong S3 bucket private.
* CloudFront phân phối website qua HTTPS.
* React Router hoạt động khi truy cập trực tiếp các URL con.
* Invalidation giúp cập nhật bản frontend mới trên toàn bộ Edge Location.
* Cognito chuyển hướng đăng nhập và đăng xuất về domain production.
* Route 53 hỗ trợ đăng ký và quản lý domain riêng.

---