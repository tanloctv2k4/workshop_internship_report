---
title : "Khởi Tạo Lambda Backend và Tích Hợp API Gateway"
date : 2026-07-14
weight : 5
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### Tổng quan

Trong phần này, bạn sẽ triển khai backend serverless cho hệ thống **PharmaCare** bằng **AWS Lambda**, công khai các chức năng thông qua **Amazon API Gateway HTTP API**, bảo vệ các API riêng tư bằng **Amazon Cognito JWT Authorizer** và kiểm tra toàn bộ luồng bằng một ứng dụng **ReactJS** chạy trên máy local.

Luồng hoạt động chính của hệ thống:

+ Người dùng đăng nhập thông qua Amazon Cognito.
+ Cognito trả về JSON Web Token (JWT) cho ứng dụng ReactJS.
+ ReactJS gửi yêu cầu HTTP đến Amazon API Gateway.
+ API Gateway kiểm tra JWT đối với các route yêu cầu đăng nhập.
+ API Gateway chuyển tiếp yêu cầu hợp lệ đến hàm Lambda `pharmacare-backend-api`.
+ Lambda đọc thông tin kết nối từ AWS Secrets Manager và truy vấn Amazon RDS PostgreSQL.

#### Điều kiện chuẩn bị

Trước khi triển khai, cần bảo đảm đã có các tài nguyên sau:

+ VPC và các Private App Subnet dành cho Lambda.
+ Amazon RDS PostgreSQL đã được khởi tạo và có thể truy cập từ Lambda.
+ AWS Secrets Manager secret chứa thông tin đăng nhập RDS.
+ IAM Role `pharmacare-lambda-role` có quyền ghi CloudWatch Logs, đọc secret và kết nối mạng trong VPC.
+ Amazon Cognito User Pool, App Client và tài khoản Customer dùng để kiểm thử.
+ Node.js, npm và Visual Studio Code trên máy local.

#### Bước 1: Tạo Lambda backend

Truy cập **AWS Lambda**, chọn **Create function** và tạo hàm mới với tên:

```text
pharmacare-backend-api
```

Chọn runtime **Node.js 22.x**, sau đó gắn hàm Lambda vào đúng VPC, Private App Subnet và Security Group đã chuẩn bị. Security Group của Lambda phải được phép kết nối đến Security Group của Amazon RDS trên cổng PostgreSQL `5432`.

![Tổng quan hàm Lambda backend](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/01-lambda-overview.png)

#### Bước 2: Cấu hình tài nguyên cho Lambda

Trong Lambda, vào **Configuration → General configuration → Edit** và cấu hình:

| Thuộc tính | Giá trị sử dụng |
|---|---:|
| Memory | `256 MB` |
| Ephemeral storage | `512 MB` |
| Timeout | `10 seconds` |
| Execution role | `pharmacare-lambda-role` |

Timeout mặc định 3 giây thường không đủ khi Lambda cần khởi tạo kết nối mạng, đọc secret và truy vấn Amazon RDS. Trong môi trường workshop, thời gian chờ 10 giây phù hợp cho quá trình kiểm thử ban đầu.

![Cấu hình Memory, Timeout và IAM Role](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/02-general-configuration.png)

#### Bước 3: Thêm Environment Variables

Vào **Configuration → Environment variables → Edit** và thêm các biến sau:

| Key | Value |
|---|---|
| `DB_NAME` | `pharmacare_ai` |
| `RDS_SECRET_ARN` | ARN của secret chứa thông tin kết nối RDS |

Không ghi trực tiếp username hoặc password của database vào mã nguồn. Lambda sẽ sử dụng `RDS_SECRET_ARN` để đọc thông tin kết nối từ AWS Secrets Manager khi hàm được thực thi.

![Cấu hình biến môi trường cho Lambda](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/03-environment-variables.png)

#### Bước 4: Tạo project backend trên Visual Studio Code

Mở Terminal trong Visual Studio Code và tạo thư mục project:

```powershell
cd D:\ThucTapTotNghiep
mkdir pharmacare-backend-api
cd pharmacare-backend-api
npm init -y
npm install pg @aws-sdk/client-secrets-manager
```

Các thư viện chính được sử dụng:

+ `pg`: tạo kết nối và truy vấn Amazon RDS PostgreSQL.
+ `@aws-sdk/client-secrets-manager`: đọc thông tin kết nối database từ AWS Secrets Manager.

![Khởi tạo project và cài đặt thư viện Node.js](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/04-backend-project-dependencies.png)

#### Bước 5: Viết mã nguồn Lambda

Tạo file `index.mjs`. Mã nguồn cần thực hiện các chức năng chính sau:

+ Đọc `AWS_REGION`, `RDS_SECRET_ARN` và `DB_NAME` từ Environment Variables.
+ Gọi AWS Secrets Manager để lấy host, port, username và password của RDS.
+ Khởi tạo PostgreSQL connection pool.
+ Phân tích HTTP method và route do API Gateway gửi đến.
+ Thực thi truy vấn tương ứng với sản phẩm, danh mục, cửa hàng, hồ sơ, giỏ hàng và đơn hàng.
+ Trả kết quả dưới dạng JSON.
+ Trả các header cần thiết cho CORS.

Ví dụ cấu trúc response:

```javascript
function response(statusCode, data) {
  return {
    statusCode,
    headers: {
      "Content-Type": "application/json",
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Headers": "Content-Type,Authorization",
      "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS"
    },
    body: JSON.stringify(data)
  };
}
```

#### Bước 6: Đóng gói mã nguồn

Đóng gói `index.mjs`, thư viện và các file package thành `function.zip`:

```powershell
Compress-Archive `
  -Path index.mjs,node_modules,package.json,package-lock.json `
  -DestinationPath function.zip `
  -Force
```

Các file phải nằm trực tiếp ở thư mục gốc của file ZIP. Không nén cả thư mục cha của project vì Lambda sẽ không tìm thấy handler.

![Đóng gói source code thành function.zip](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/05-package-lambda-code.png)

#### Bước 7: Tải mã nguồn lên Lambda

Trong tab **Code** của Lambda:

1. Chọn **Upload from → .zip file**.
2. Chọn file `function.zip`.
3. Chờ quá trình tải lên hoàn tất.
4. Kiểm tra Runtime là **Node.js 22.x**.
5. Kiểm tra Handler là `index.handler`.
6. Chọn **Deploy** để áp dụng phiên bản mã nguồn mới.

![Mã nguồn backend sau khi tải lên Lambda](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/06-upload-lambda-code.png)

#### Bước 8: Tạo Amazon API Gateway HTTP API

Truy cập **Amazon API Gateway → Create API** và chọn loại **HTTP API**. HTTP API được sử dụng để tạo URL public cho frontend ReactJS gọi đến Lambda.

![Chọn loại Amazon API Gateway HTTP API](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/07-select-http-api.png)

Trong phần **Integrations**, chọn Lambda function `pharmacare-backend-api` làm integration target.

![Tích hợp HTTP API với Lambda backend](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/08-configure-api-integration.png)

#### Bước 9: Tạo các route public

Khai báo các route không yêu cầu đăng nhập:

| Method | Resource path | Chức năng |
|---|---|---|
| `GET` | `/health` | Kiểm tra trạng thái backend |
| `GET` | `/products` | Lấy danh sách sản phẩm |
| `GET` | `/products/{id}` | Lấy chi tiết sản phẩm |
| `GET` | `/categories` | Lấy danh sách danh mục |
| `GET` | `/stores` | Lấy danh sách cửa hàng |

Tất cả các route trên sử dụng integration target `pharmacare-backend-api`.

![Khai báo các route public](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/09-configure-public-routes.png)

#### Bước 10: Cấu hình Stage

Tại bước **Define stages**, sử dụng stage `$default` và bật **Auto-deploy**. Khi route hoặc integration được thay đổi, cấu hình mới sẽ tự động được triển khai mà không cần tạo deployment thủ công.

![Cấu hình stage mặc định cho HTTP API](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/10-define-default-stage.png)

Kiểm tra lại integration, route và stage trước khi chọn **Create**.

![Kiểm tra cấu hình trước khi tạo API](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/11-review-create-api.png)

Sau khi API được tạo, kiểm tra danh sách route trong mục **Develop → Routes**.

![Danh sách route sau khi tạo HTTP API](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/12-api-routes-created.png)

#### Bước 11: Cấu hình CORS

Vào **Develop → CORS** và nhập cấu hình dùng cho frontend local:

| Thuộc tính | Giá trị |
|---|---|
| Allow origins | `http://localhost:5173`, `http://localhost:3000` |
| Allow headers | `content-type`, `authorization` |
| Allow methods | `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS` |
| Max age | `3600` |
| Allow credentials | `No` |

`localhost:5173` là địa chỉ mặc định của Vite. `localhost:3000` được giữ lại để hỗ trợ trường hợp frontend chạy bằng công cụ khác.

![Cấu hình CORS cho ReactJS](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/13-configure-cors.png)

#### Bước 12: Tạo Cognito JWT Authorizer

Vào **Develop → Authorization → Manage authorizers → Create** và chọn loại **JWT**.

Cấu hình authorizer:

| Thuộc tính | Giá trị |
|---|---|
| Name | `pharmacare-cognito-authorizer` |
| Identity source | `$request.header.Authorization` |
| Issuer URL | `https://cognito-idp.ap-southeast-1.amazonaws.com/<USER_POOL_ID>` |
| Audience | `<APP_CLIENT_ID>` |

Trong đó:

+ `<USER_POOL_ID>` là User Pool ID của Cognito.
+ `<APP_CLIENT_ID>` là Client ID của ứng dụng frontend.

Không đưa Client Secret vào frontend ReactJS. Đối với ứng dụng Single Page Application, App Client nên được cấu hình không sử dụng client secret.

![Tạo Cognito JWT Authorizer](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/14-create-jwt-authorizer.png)

Sau khi tạo thành công, authorizer sẽ xuất hiện trong danh sách quản lý của API Gateway.

![JWT Authorizer sau khi được tạo](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/15-authorizer-created.png)

#### Bước 13: Gắn Authorizer vào các route riêng tư

Các route public như `/health`, `/products`, `/categories` và `/stores` không cần authorizer. Các route liên quan đến dữ liệu cá nhân, giỏ hàng và đơn hàng phải yêu cầu JWT.

Các route cần gắn `pharmacare-cognito-authorizer`:

| Method | Resource path |
|---|---|
| `GET` | `/profile` |
| `GET` | `/cart` |
| `POST` | `/cart` |
| `PUT` | `/cart/{productId}` |
| `DELETE` | `/cart/{productId}` |
| `POST` | `/orders` |
| `GET` | `/orders` |
| `GET` | `/orders/{id}` |

Đối với từng route, chọn **Attach authorizer**, chọn `pharmacare-cognito-authorizer` và lưu cấu hình.

![Gắn Cognito Authorizer vào route GET profile](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/16-attach-authorizer-route.png)

#### Bước 14: Kiểm tra đăng nhập bằng Cognito

Sử dụng tài khoản Customer đã tạo trong Cognito để đăng nhập. Sau khi nhập đúng thông tin, Cognito chuyển người dùng trở lại địa chỉ callback của frontend cùng trạng thái xác thực.

![Kiểm tra đăng nhập bằng tài khoản Cognito](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/17-cognito-login-test.png)

#### Bước 15: Tạo frontend ReactJS để kiểm tra API

Mở Terminal trong Visual Studio Code và chạy:

```powershell
cd D:\ThucTapTotNghiep
npm create vite@latest pharmacare-frontend -- --template react
cd pharmacare-frontend
npm install
npm install react-oidc-context
npm run dev
```

![Tạo ứng dụng ReactJS bằng Vite](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/18-create-react-vite.png)

Sau khi Vite khởi động, truy cập:

```text
http://localhost:5173
```

![Ứng dụng ReactJS chạy trên máy local](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/19-react-development-server.png)

#### Bước 16: Tạo giao diện kiểm thử

Trong `App.jsx`, tạo giao diện đơn giản gồm:

+ Nút đăng nhập và đăng xuất Cognito.
+ Nhóm nút gọi các public API.
+ Nhóm nút gọi các protected API.
+ Khu vực hiển thị status code và JSON response.

Khi gọi protected API, frontend phải gửi token trong header:

```http
Authorization: Bearer <access_token>
```

![Code giao diện ReactJS dùng để kiểm thử](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/20-react-test-interface-code.png)

#### Bước 17: Kiểm tra kết quả

Thực hiện các kịch bản kiểm thử sau:

| Kịch bản | Kết quả mong đợi |
|---|---|
| Gọi `GET /health` khi chưa đăng nhập | API trả trạng thái hoạt động bình thường |
| Gọi `GET /products` khi chưa đăng nhập | Nhận danh sách sản phẩm |
| Gọi `GET /profile` khi chưa đăng nhập | API Gateway từ chối do thiếu JWT |
| Đăng nhập Cognito rồi gọi `GET /profile` | Nhận thông tin người dùng |
| Gọi các route `/cart` và `/orders` với JWT hợp lệ | Lambda xử lý yêu cầu và truy vấn RDS |
| Gửi JWT hết hạn hoặc sai audience | API Gateway trả lỗi xác thực |

Giao diện kiểm thử hiển thị trạng thái đăng nhập, các nút gọi API và dữ liệu JSON trả về từ backend.

![Kết quả kiểm thử frontend, API Gateway và Lambda](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/21-api-test-result.png)

#### Kết quả đạt được

Sau khi hoàn thành, hệ thống đã xây dựng được luồng backend serverless gồm:

+ ReactJS làm ứng dụng frontend.
+ Amazon Cognito quản lý đăng nhập và cấp JWT.
+ Amazon API Gateway công khai HTTP API và kiểm tra quyền truy cập.
+ AWS Lambda xử lý nghiệp vụ backend.
+ AWS Secrets Manager bảo vệ thông tin kết nối.
+ Amazon RDS PostgreSQL lưu trữ dữ liệu PharmaCare.

Các API public có thể được sử dụng mà không cần đăng nhập. Các API liên quan đến hồ sơ, giỏ hàng và đơn hàng chỉ được thực thi khi yêu cầu chứa JWT hợp lệ.

#### Lưu ý bảo mật

Trước khi đưa ảnh lên website công khai, cần che hoặc thay thế các thông tin nhận dạng tài nguyên như AWS Account ID, Function ARN, Secret ARN, API ID, User Pool ID, App Client ID và URL callback thực tế. Không đăng password database, access key, secret access key hoặc token người dùng lên báo cáo.