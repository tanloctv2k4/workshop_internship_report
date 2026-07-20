---
title : "Lưu Trữ S3 Và Phân Phối CDN CloudFront"
date : 2026-07-18
weight : 6
chapter : false
pre : " <b> 5.4.2 </b> "
---

## 1. Khởi Tạo Amazon S3 Bucket Lưu Trữ Ảnh Sản Phẩm

### Các bước thực hiện:
1. Mở [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1), trong thanh điều hướng bên trái chọn **Buckets**, sau đó nhấn nút **Create bucket**.
2. Cấu hình các thông số chi tiết cho Bucket lưu trữ:
   * **AWS Region:** Chọn `Asia Pacific (Singapore) ap-southeast-1` (Đảm bảo độ trễ thấp nhất và đồng bộ khu vực với hạ tầng VPC, RDS của hệ thống PharmaCare).
   * **Bucket type:** Chọn **General purpose** (Khuyến nghị cho hầu hết các mô hình lưu trữ đối tượng tiêu chuẩn).
   * **Bucket name:** `pharmacare-product-images-phu-2026` (Lưu ý: Tên bucket phải là duy nhất trên toàn cầu, định dạng chữ thường, số và dấu gạch ngang).
   * **Object Ownership:** Chọn **ACLs disabled (recommended)** -> Hệ thống tự động áp dụng chính sách **Bucket owner enforced**, toàn bộ quyền sở hữu đối tượng thuộc về tài khoản AWS và được kiểm soát tập trung qua IAM / Bucket Policy.
   * **Block Public Access settings for this bucket:** Tích chọn **Block all public access** (Đảm bảo an toàn bảo mật tuyệt đối, ngăn chặn việc truy cập trái phép trực tiếp từ Internet vào S3; luồng truy cập public sẽ được cấp quyền đi qua cổng CDN).
   * **Bucket Versioning:** Chọn **Disable** (Tối ưu hóa chi phí lưu trữ đối với các tài nguyên ảnh tĩnh của sản phẩm ít thay đổi).
   * **Default encryption:** Chọn **Server-side encryption with Amazon S3 managed keys (SSE-S3)** và bật **Bucket Key: Enable** để tự động mã hóa dữ liệu ở trạng thái nghỉ (data at rest), đồng thời tối ưu chi phí gọi lệnh mã hóa.
3. Kiểm tra lại toàn bộ cấu hình trực quan và nhấn nút **Create bucket** để hoàn tất khởi tạo.

   ![Khởi tạo S3 Bucket cho PharmaCare](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket1.jpg)

Việc thiết lập S3 Bucket với quy tắc bảo mật **Block all public access** là tiêu chuẩn bắt buộc trong kiến trúc Cloud Native của PharmaCare. Thay vì mở công khai bucket ra Internet — tiềm ẩn rủi ro tấn công DDoS, rà quét trái phép và chi phí phát sinh truyền tải lớn — hệ thống đóng kín lưu trữ gốc và chỉ phân quyền truy cập thông qua mạng lưới phân phối nội dung (CloudFront CDN) bằng các cơ chế bảo mật tự động của AWS.

---

## 2. Tạo Cấu Trúc Thư Mục Và Tải Ảnh Sản Phẩm Lên S3

### Các bước thực hiện:
1. Trong danh sách **Buckets**, nhấp vào tên bucket vừa khởi tạo: `pharmacare-product-images-phu-2026`.
2. Tại giao diện tab **Objects**, click **Create folder** để tạo cấu trúc phân cấp quản lý tệp tin.
3. Cấu hình thông tin thư mục:
   * **Folder name:** `products` (Thiết lập đường dẫn gốc chuẩn hóa cho tài nguyên hình ảnh sản phẩm).
   * **Server-side encryption:** Chọn **Don't specify an encryption key** (Thư mục sẽ tự động kế thừa cấu hình mã hóa mặc định SSE-S3 từ Bucket).
   * Nhấn nút **Create folder** để xác nhận.

   ![Tạo thư mục products trong S3](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket2.jpg)

4. Khi thư mục `products/` xuất hiện trong danh sách đối tượng với trạng thái sẵn sàng, click trực tiếp vào thư mục để tiến vào không gian lưu trữ bên trong.

   ![Danh sách đối tượng trong bucket](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket3.jpg)

5. Nhấn nút **Upload**, tiếp tục chọn **Add files** và chọn tập hợp các tệp ảnh sản phẩm AI của PharmaCare (từ `PC-0001.png` đến `PC-0016.png` cùng các tệp liên quan, tổng cộng 193 đối tượng).
6. Giữ nguyên hạng mục lưu trữ ở mức **Standard** và nhấn **Upload** để thực hiện truyền tải dữ liệu lên đám mây.

   ![Tải ảnh sản phẩm vào thư mục products](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket4.jpg)

Mặc dù Amazon S3 lưu trữ dữ liệu dưới dạng cấu trúc phẳng (flat storage schema), việc sử dụng tiền tố thư mục (prefix) `products/` giúp tổ chức không gian định danh một cách logic, dễ dàng phân loại tài nguyên, áp dụng các chính sách vòng đời (Lifecycle rules) và hỗ trợ định tuyến chính xác trên CDN khi mở rộng quy mô hệ thống sau này.

---

## 3. Khởi Tạo Mạng Lưới Phân Phối Nội Dung (Amazon CloudFront CDN)

### Các bước thực hiện:
1. Mở [Amazon CloudFront console](https://console.aws.amazon.com/cloudfront/v3/home), điều hướng đến mục **Distributions** và nhấn chọn **Create distribution**.
2. **Bước 1 - Choose a plan:** Chọn gói **Free ($0/month)** được cung cấp cho mục đích học tập và phát triển hệ thống, tích hợp sẵn các bảo vệ DDoS (Always-on DDoS protection), chứng chỉ TLS miễn phí, Tiered caching, cùng định mức 1M requests / 100GB lưu lượng hàng tháng. Nhấn **Next**.

   ![Chọn gói CloudFront Free Plan](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket5.jpg)

3. **Bước 2 - Get started (Distribution options):**
   * **Distribution name:** `pharmacare-product-images-cdn` (Định danh hệ thống mạng lưới CDN).
   * **Description:** `CloudFront CDN for PharmaCare AI product images from S3`.
   * **Distribution type:** Chọn **Single website or app** (Tối ưu hóa cấu hình cho kiến trúc ứng dụng độc lập).
   * Nhấn **Next**.

   ![Cấu hình thông tin cơ bản CloudFront](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket6.jpg)

4. **Bước 3 - Specify origin:**
   * **Origin type:** Chọn **Amazon S3**.
   * **Origin:** Chọn S3 bucket gốc đã khởi tạo trước đó: `pharmacare-product-images-phu-2026.s3.ap-southeast-1.amazonaws.com`.
   * **Allow private S3 bucket access to CloudFront:** Tích chọn **Allow private S3 bucket access to CloudFront - Recommended** (Tính năng này tự động cấp quyền cho CloudFront thông qua việc cập nhật S3 Bucket Policy, đảm bảo duy nhất CloudFront có quyền truy cập xuất bản nội dung từ bucket kín).
   * **Origin settings:** Chọn **Use recommended origin settings**.
   * **Cache settings:** Chọn **Use recommended cache settings tailored to serving S3 content** (Tối ưu bộ nhớ đệm cho tệp tĩnh).
   * Nhấn **Next**.

   ![Cấu hình Origin S3 cho CloudFront](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket7.jpg)

5. **Bước 4 - Enable security:**
   * Tại mục **Web Application Firewall (WAF)**, chế độ bảo mật tích hợp sẵn (**Included security protections**) được tự động kích hoạt để bảo vệ ứng dụng trước các lỗ hổng web phổ biến, ngăn chặn các tác nhân rà quét độc hại và chặn các IP thuộc danh sách đe dọa (Threat intelligence).
   * Nhấn **Next**.

   ![Kích hoạt bảo mật WAF tích hợp](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket8.jpg)

6. **Bước 5 - Review and create:**
   * Kiểm tra tổng thể các thông số cấu hình. Ghi nhận thông báo hệ thống: *"Because you granted CloudFront access to your origin, CloudFront can write and update S3 bucket policies that restrict access to your S3 origin to CloudFront"*.
   * Nhấn nút **Create distribution** để kích hoạt tiến trình triển khai mạng lưới biên toàn cầu.

   ![Xác nhận và tạo CloudFront Distribution](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket9.jpg)

Amazon CloudFront đóng vai trò là mạng phân phối nội dung toàn cầu với hàng trăm máy chủ biên (Edge Locations). Khi người dùng cuối truy cập giao diện PharmaCare, hình ảnh sản phẩm sẽ được phục vụ trực tiếp từ máy chủ biên gần nhất thay vì phải gửi yêu cầu về S3 gốc tại Singapore, mang lại các lợi ích vượt trội:
* **Giảm tối đa độ trễ (Latency):** Tăng tốc tải trang gấp nhiều lần, nâng cao trải nghiệm mượt mà cho người dùng.
* **Tối ưu chi phí truyền tải (Cost Efficiency):** Tận dụng bộ nhớ đệm tại biên (caching) để giảm tải số lượng request đọc trực tiếp vào S3 Bucket gốc.
* **Tăng cường bảo mật đa lớp (Security & WAF):** Che giấu hoàn toàn địa chỉ S3 thực tế, đồng thời ngăn chặn hiệu quả các cuộc tấn công DDoS ở tầng mạng (Layer 3/4) và tầng ứng dụng (Layer 7).

---

## 4. Kiểm Tra Trạng Thái Kích Hoạt CloudFront Distribution

### Các bước thực hiện:
1. Tại trang quản trị chi tiết của Distribution vừa tạo, theo dõi tiến trình triển khai cho đến khi trạng thái `Deploying` chuyển hoàn toàn sang `Enabled` và hiển thị thông báo thành công: **Successfully created new distribution**.
2. Ghi nhận các thông số mạng quan trọng để phục vụ bước cấu hình tích hợp cơ sở dữ liệu ứng dụng:
   * **Distribution domain name:** `d2sq4q0ymcz8do.cloudfront.net` (Tên miền công khai chính thức sử dụng để truy cập tài nguyên ảnh sản phẩm).
   * **ARN:** `arn:aws:cloudfront::392646748514:distribution/E105N1I4TWNVYQ`.

   ![Trạng thái triển khai CloudFront thành công](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket10.jpg)

3. **Kiểm tra nhanh (Smoke Test):** Mở trình duyệt web và truy cập thử vào đường dẫn hình ảnh thực tế theo định dạng tên miền CDN:
   `https://d2sq4q0ymcz8do.cloudfront.net/products/PC-0001.png` để xác nhận tài nguyên được phục vụ thành công qua CloudFront mà không cần mở quyền truy cập public trên S3.

---

## 5. Cập Nhật Đường Dẫn CDN Vào Cơ Sở Dữ Liệu Amazon RDS (Thông Qua AWS Lambda)

### Các bước thực hiện:
1. Mở môi trường phát triển tích hợp (Visual Studio Code), truy cập vào thư mục mã nguồn dự án `pharmacare-db-migration`.
2. Chỉnh sửa tệp logic xử lý migration (`index.mjs`) để tự động hóa việc cập nhật tên miền CloudFront mới vào cột `image_url` của bảng `products` trong cơ sở dữ liệu Amazon RDS.
   * **Đoạn mã JavaScript xử lý cập nhật (`index.mjs`):**
     ```javascript
     import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";
     import pg from "pg";
     const { Client } = pg;
     
     const REGION = process.env.AWS_REGION || "ap-southeast-1";
     const SECRET_ARN = process.env.RDS_SECRET_ARN;
     const DB_NAME = process.env.DB_NAME || "pharmacare_ai";
     const DB_HOST = process.env.DB_HOST;
     const DB_PORT = Number(process.env.DB_PORT || 5432);

     // Câu lệnh SQL cập nhật đường dẫn hình ảnh sử dụng CloudFront Domain
     const migrationSQL = `
       UPDATE products
       SET image_url = '[https://d2sq4q0ymcz8do.cloudfront.net/products/](https://d2sq4q0ymcz8do.cloudfront.net/products/)' || sku || '.png',
           updated_at = CURRENT_TIMESTAMP
       WHERE sku IS NOT NULL;
     `;

     export const handler = async () => {
       if (!SECRET_ARN) {
         throw new Error("Missing environment variable: RDS_SECRET_ARN");
       }
       
       const secretsClient = new SecretsManagerClient({ region: REGION });
       const secretResponse = await secretsClient.send(
         new GetSecretValueCommand({
           SecretId: SECRET_ARN
         })
       );
       // Logic kết nối DB và thực thi lệnh migrationSQL...
     };
     ```

   ![Chỉnh sửa mã nguồn migration trong VS Code](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket11.jpg)

3. Đóng gói tệp mã nguồn cùng các thư viện phụ thuộc (`node_modules`) thành tệp nén `function.zip` thông qua lệnh Terminal (PowerShell):
   ```powershell
   Compress-Archive -Path index.mjs,package.json,package-lock.json,node_modules -DestinationPath function.zip -Force
   # Cập nhật và triển khai hàm AWS Lambda Migration

4. Mở [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions), tìm kiếm và chọn hàm serverless chịu trách nhiệm thực hiện quá trình migration cơ sở dữ liệu: `pharmacare-db-migration`.

5. Trong giao diện **Code source**, nhấn **Upload from** → **.zip file**, sau đó tải lên file `function.zip` vừa được đóng gói.

   Ngoài ra, có thể sử dụng trực tiếp tiện ích mở rộng **AWS Toolkit** trong Visual Studio Code bằng cách nhấn **Deploy (Ctrl+Shift+U)** để triển khai mã nguồn lên AWS Lambda.

   ![Cập nhật và triển khai mã nguồn lên AWS Lambda](/workshop_internship_report/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket12.jpg)

6. Thực thi chức năng **Test** của Lambda để kích hoạt lệnh migration SQL. Kiểm tra nhật ký thực thi trong **Amazon CloudWatch Logs** nhằm xác nhận rằng tất cả bản ghi sản phẩm trong Amazon RDS đã được cập nhật thành công trường `image_url`, trỏ đến endpoint CDN của CloudFront.

## Tự động hóa an toàn bằng AWS Lambda

Thay vì kết nối thủ công từ máy tính cá nhân đến cơ sở dữ liệu production — phương pháp có thể gây ra rủi ro vận hành và làm lộ thông tin xác thực — việc sử dụng AWS Lambda kết hợp với **AWS Secrets Manager** thông qua biến `RDS_SECRET_ARN` giúp quá trình migration được thực hiện trong một môi trường mạng cô lập, được mã hóa đầy đủ và có thể kiểm tra, giám sát thông qua CloudWatch.

## Hoàn thiện kiến trúc Cloud Native

Sau bước này, toàn bộ dữ liệu có cấu trúc như metadata, giá sản phẩm và tên sản phẩm được lưu trữ an toàn trong Amazon RDS thuộc các **Private Subnet**. Trong khi đó, các tài nguyên phi cấu trúc có dung lượng lớn như hình ảnh sản phẩm được phân phối với tốc độ cao thông qua mạng lưới edge của CloudFront.

Điều này giúp hoàn thiện một kiến trúc PharmaCare được tối ưu về hiệu năng, chi phí và bảo mật.