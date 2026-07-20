---
title : "Giám Sát, Sao Lưu Và Bảo Mật Hệ Thống PharmaCare"
date : 2026-07-19
weight : 9
chapter : false
pre : " <b> 5.7 </b> "
---

## 1. Tổng Quan Giám Sát, Sao Lưu Và Bảo Mật

Sau khi hoàn tất triển khai frontend, backend và chatbot AI, hệ thống PharmaCare cần được bổ sung các cơ chế vận hành nhằm bảo đảm tính ổn định, khả năng phục hồi dữ liệu và an toàn khi cung cấp dịch vụ trên Internet.

Các thành phần được triển khai trong phần này gồm:

* **Amazon CloudWatch Logs:** Thu thập và lưu trữ nhật ký thực thi của các Lambda function.
* **Amazon SNS:** Gửi thông báo cảnh báo đến quản trị viên.
* **Amazon CloudWatch Alarm:** Theo dõi lỗi backend, chatbot, API Gateway và mức sử dụng CPU của RDS.
* **AWS Backup:** Tự động sao lưu cơ sở dữ liệu Amazon RDS.
* **AWS WAF:** Bảo vệ CloudFront trước các request độc hại và lỗ hổng web phổ biến.
* **AWS IAM:** Cấp quyền theo nguyên tắc đặc quyền tối thiểu cho các Lambda function.

Luồng giám sát và cảnh báo tổng thể:

```text
Lambda, API Gateway và Amazon RDS
        ↓
CloudWatch Logs và CloudWatch Metrics
        ↓
CloudWatch Alarm
        ↓
Amazon SNS Topic
        ↓
Email của quản trị viên
```

Luồng bảo vệ và phục hồi dữ liệu:

```text
Người dùng Internet
        ↓
Amazon CloudFront
        ↓
AWS WAF
        ↓
Frontend PharmaCare

Amazon RDS
        ↓
AWS Backup Plan
        ↓
Backup Vault
        ↓
Recovery Point
```

---

## 2. Kiểm Tra CloudWatch Log Groups

Amazon CloudWatch Logs tự động lưu nhật ký thực thi của AWS Lambda khi execution role có quyền `AWSLambdaBasicExecutionRole`.

### Các bước thực hiện:

1. Mở [Amazon CloudWatch Console](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1).
2. Trong thanh điều hướng bên trái, chọn **Logs** → **Log management**.
3. Mở tab **Log groups**.
4. Kiểm tra các log group của hệ thống PharmaCare:

   ```text
   /aws/lambda/pharmacare-backend-api
   /aws/lambda/pharmacare-db-migration
   /aws/lambda/pharmacare-rag-chatbot
   /aws/lambda/pharmacare-rag-indexing
   ```

   ![Danh sách CloudWatch Log Groups của PharmaCare](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs1.jpg)

Mỗi Lambda function có một log group riêng. Bên trong log group, CloudWatch tạo các log stream tương ứng với từng execution environment.

### Nội dung cần theo dõi

* Thời gian bắt đầu và kết thúc request.
* Request ID của Lambda.
* Thời gian thực thi.
* Bộ nhớ đã sử dụng.
* Lỗi kết nối Amazon RDS.
* Lỗi truy cập AWS Secrets Manager.
* Lỗi gọi Amazon Bedrock.
* Lỗi DNS hoặc timeout khi truy cập OpenSearch Serverless.
* Lỗi xử lý câu hỏi từ frontend.
* Kết quả indexing tài liệu.

### Cấu hình thời gian lưu log

Trong ảnh, các log group đang sử dụng giá trị:

```text
Retention: Never expire
```

Để tối ưu chi phí, nên cấu hình thời gian lưu phù hợp với môi trường:

| Môi trường | Retention đề xuất |
|---|---|
| Development | 7 hoặc 14 ngày |
| Testing | 14 hoặc 30 ngày |
| Production | 30, 60 hoặc 90 ngày |
| Log phục vụ kiểm toán | Theo chính sách của tổ chức |

Các bước thay đổi retention:

1. Chọn log group.
2. Chọn **Actions**.
3. Chọn **Edit retention setting**.
4. Chọn số ngày lưu.
5. Nhấn **Save**.

---

## 3. Tạo Amazon SNS Topic Gửi Cảnh Báo

Amazon Simple Notification Service được sử dụng làm kênh gửi thông báo khi CloudWatch Alarm chuyển sang trạng thái `ALARM`.

### Các bước thực hiện:

1. Mở [Amazon SNS Console](https://ap-southeast-1.console.aws.amazon.com/sns/v3/home?region=ap-southeast-1).
2. Chọn **Topics**.
3. Nhấn **Create topic**.
4. Cấu hình:

   * **Type:** `Standard`.
   * **Name:** `pharmacare-alert-topic`.
   * **Display name:** `Pharmacare Alert`.

5. Nhấn **Create topic**.
6. Kiểm tra topic được tạo thành công.

   ![Tạo SNS Topic gửi cảnh báo PharmaCare](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs2.jpg)

SNS Topic đóng vai trò là điểm nhận thông báo tập trung. Nhiều CloudWatch Alarm có thể cùng gửi cảnh báo đến một topic, giúp quản trị viên không phải cấu hình email riêng cho từng alarm.

---

## 4. Tạo Email Subscription Cho SNS Topic

Sau khi tạo topic, cần đăng ký email của quản trị viên để nhận cảnh báo.

### Các bước thực hiện:

1. Mở topic:

   ```text
   pharmacare-alert-topic
   ```

2. Chọn **Create subscription**.
3. Cấu hình:

   * **Protocol:** `Email`.
   * **Endpoint:** Nhập email của quản trị viên.

4. Nhấn **Create subscription**.
5. Kiểm tra trạng thái subscription:

   ```text
   Pending confirmation
   ```

   ![Tạo Email Subscription cho SNS Topic](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs3.jpg)

### Xác nhận subscription

SNS chưa gửi cảnh báo khi subscription còn ở trạng thái `Pending confirmation`.

Quản trị viên cần:

1. Mở hộp thư email đã đăng ký.
2. Tìm email có tiêu đề xác nhận từ AWS Notifications.
3. Nhấn liên kết **Confirm subscription**.
4. Quay lại Amazon SNS Console.
5. Kiểm tra trạng thái chuyển sang:

   ```text
   Confirmed
   ```

> Nếu không tìm thấy email, cần kiểm tra thư mục Spam, Promotions hoặc yêu cầu gửi lại xác nhận.

### Kiểm tra hoạt động của SNS

Sau khi subscription được xác nhận:

1. Mở `pharmacare-alert-topic`.
2. Nhấn **Publish message**.
3. Nhập tiêu đề và nội dung kiểm thử.
4. Gửi message.
5. Kiểm tra email quản trị viên đã nhận thông báo.

---

## 5. Tạo CloudWatch Alarm Cho Hệ Thống

CloudWatch Alarm theo dõi metric của các dịch vụ AWS và kích hoạt hành động khi metric vượt ngưỡng đã cấu hình.

Hệ thống PharmaCare sử dụng bốn alarm:

```text
pharmacare-rds-cpu-high-alarm
pharmacare-api-5xx-alarm
pharmacare-chatbot-errors-alarm
pharmacare-backend-errors-alarm
```

   ![Danh sách CloudWatch Alarm của PharmaCare](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs4.jpg)

### 5.1. Alarm CPU Cao Cho Amazon RDS

Tên alarm:

```text
pharmacare-rds-cpu-high-alarm
```

Điều kiện trong hệ thống:

```text
CPUUtilization > 80
trong 2 datapoints
trong khoảng thời gian 10 phút
```

Mục đích:

* Phát hiện database bị quá tải CPU.
* Phát hiện truy vấn SQL chưa tối ưu.
* Phát hiện Lambda tạo quá nhiều kết nối đồng thời.
* Cảnh báo khi cấu hình RDS không còn đủ tài nguyên.

Khi alarm kích hoạt, cần kiểm tra:

* CloudWatch Metrics của RDS.
* Số lượng Database Connections.
* Freeable Memory.
* Read/Write Latency.
* Slow query hoặc truy vấn giữ khóa lâu.
* Số lần Lambda gọi database.

### 5.2. Alarm Lỗi 5xx Của API

Tên alarm:

```text
pharmacare-api-5xx-alarm
```

Điều kiện:

```text
5XXError >= 1
trong 1 datapoint
trong khoảng thời gian 5 phút
```

Alarm này phát hiện các lỗi phía server được trả về từ API Gateway hoặc backend integration.

Các nguyên nhân thường gặp:

* Lambda throw exception.
* Lambda timeout.
* Lỗi kết nối RDS.
* Sai cấu hình integration response.
* Lambda thiếu quyền IAM.
* Backend trả dữ liệu không đúng định dạng.

### 5.3. Alarm Lỗi Chatbot Lambda

Tên alarm:

```text
pharmacare-chatbot-errors-alarm
```

Điều kiện:

```text
Errors >= 1
trong 1 datapoint
trong khoảng thời gian 5 phút
```

Alarm theo dõi Lambda:

```text
pharmacare-rag-chatbot
```

Các lỗi cần chú ý:

* Bedrock model access bị từ chối.
* OpenSearch endpoint không truy cập được.
* Index không tồn tại.
* Vector dimension không khớp.
* Lambda timeout.
* Câu hỏi đầu vào không hợp lệ.
* Thiếu Environment Variable.

### 5.4. Alarm Lỗi Backend Lambda

Tên alarm:

```text
pharmacare-backend-errors-alarm
```

Điều kiện:

```text
Errors >= 1
trong 1 datapoint
trong khoảng thời gian 5 phút
```

Alarm này giám sát Lambda backend xử lý sản phẩm, tài khoản, đơn hàng và tồn kho.

### 5.5. Gắn SNS Topic Vào Alarm

Trong bước cấu hình notification:

1. Chọn **In alarm**.
2. Chọn **Select an existing SNS topic**.
3. Chọn:

   ```text
   pharmacare-alert-topic
   ```

4. Hoàn tất tạo alarm.

Khi metric vượt ngưỡng:

```text
CloudWatch Metric
        ↓
Alarm chuyển sang ALARM
        ↓
SNS pharmacare-alert-topic
        ↓
Email quản trị viên
```

### Trạng thái của Alarm

* **OK:** Metric đang nằm trong giới hạn an toàn.
* **ALARM:** Metric đã vượt ngưỡng.
* **Insufficient data:** CloudWatch chưa có đủ dữ liệu để đánh giá.

Trạng thái `Insufficient data` có thể xuất hiện khi Lambda chưa được gọi hoặc metric mới được tạo.

---

## 6. Tạo AWS Backup Plan Cho Amazon RDS

AWS Backup được sử dụng để tự động hóa quá trình sao lưu database PharmaCare theo lịch định kỳ.

### Các bước thực hiện:

1. Mở [AWS Backup Console](https://ap-southeast-1.console.aws.amazon.com/backup/home?region=ap-southeast-1).
2. Chọn **Backup plans**.
3. Nhấn **Create backup plan**.
4. Chọn tạo plan mới.
5. Cấu hình:

   * **Backup plan name:** `pharmacare-rds-backup-plan`.
   * **Backup rule name:** `daily-rds-backup`.
   * **Backup vault:** `Default`.
   * **Backup frequency:** Hằng ngày.
   * **Backup window:** Chọn thời gian ít người dùng.
   * **Lifecycle:** Cấu hình thời gian lưu theo nhu cầu.

6. Hoàn tất tạo backup plan.

   ![AWS Backup Plan cho Amazon RDS](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs5.jpg)

Backup plan trong hệ thống có một rule:

```text
daily-rds-backup
```

Rule này xác định:

* Lịch chạy backup.
* Khoảng thời gian bắt đầu backup.
* Thời gian cho phép backup hoàn thành.
* Backup Vault đích.
* Chính sách vòng đời của recovery point.

### Retention đề xuất

| Môi trường | Retention tham khảo |
|---|---|
| Development | 7 ngày |
| Testing | 7–14 ngày |
| Production | 30 ngày hoặc lâu hơn |
| Dữ liệu quan trọng | Theo yêu cầu phục hồi và kiểm toán |

Cần cân đối giữa thời gian lưu, yêu cầu khôi phục và chi phí lưu trữ.

---

## 7. Gán Amazon RDS Vào Backup Plan

Sau khi tạo backup plan, cần chỉ định database được bảo vệ.

### Các bước thực hiện:

1. Mở backup plan:

   ```text
   pharmacare-rds-backup-plan
   ```

2. Chọn **Assign resources**.
3. Cấu hình:

   * **Resource assignment name:** `pharmacare-rds-resource-assignment`.
   * **IAM role:** `AWSBackupDefaultServiceRole`.
   * **Resource type:** Amazon RDS.
   * **Resource:** Database `pharmacare-db`.

4. Lưu cấu hình resource assignment.

   ![Gán Amazon RDS vào AWS Backup Plan](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs6.jpg)

Resource ID trong cấu hình có dạng:

```text
arn:aws:rds:ap-southeast-1:<account-id>:db:pharmacare-db
```

### Kiểm tra backup

1. Mở **Jobs** → **Backup jobs**.
2. Kiểm tra trạng thái job:

   ```text
   Completed
   ```

3. Mở **Vaults** → `Default`.
4. Kiểm tra recovery point của RDS.
5. Ghi nhận:

   * Thời gian tạo.
   * Resource ARN.
   * Expiration date.
   * Backup size.
   * Encryption key.

### Kiểm thử phục hồi

Một hệ thống backup chỉ được xem là đáng tin cậy khi đã kiểm thử restore.

Quy trình kiểm thử:

1. Chọn recovery point.
2. Nhấn **Restore**.
3. Khôi phục thành một RDS instance mới.
4. Không ghi đè trực tiếp database production.
5. Kết nối vào database được phục hồi.
6. Kiểm tra bảng, số lượng bản ghi và dữ liệu đơn hàng.
7. Xóa tài nguyên kiểm thử sau khi hoàn tất để tránh phát sinh chi phí.

---

## 8. Bật AWS WAF Cho CloudFront

AWS WAF bảo vệ website trước các request web độc hại trước khi chúng được chuyển đến CloudFront origin.

### Các bước thực hiện:

1. Mở [Amazon CloudFront Console](https://console.aws.amazon.com/cloudfront/v3/home).
2. Chọn Distribution:

   ```text
   pharmacare-frontend-distribution
   ```

3. Chọn tab **Security**.
4. Mở phần **Web Application Firewall (WAF)**.
5. Kiểm tra **Core protections** ở trạng thái:

   ```text
   Enabled
   ```

6. Trong giai đoạn đầu, có thể chạy ở **Monitor mode** để theo dõi request trước khi áp dụng chặn.

   ![AWS WAF được gắn với CloudFront Distribution](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs7.jpg)

### Vai trò của AWS WAF

* Phát hiện request bất thường.
* Giảm nguy cơ khai thác lỗ hổng web phổ biến.
* Chặn IP hoặc request pattern độc hại.
* Hỗ trợ rate limiting.
* Bảo vệ đường dẫn frontend và các tài nguyên phân phối qua CloudFront.
* Ghi nhận số request được cho phép hoặc bị chặn.

### Chuyển từ Monitor sang Block

Trong giai đoạn kiểm thử:

```text
Rule action: Count hoặc Monitor
```

Sau khi xác nhận rule không chặn nhầm người dùng hợp lệ, có thể chuyển sang:

```text
Rule action: Block
```

Không nên bật hàng loạt rule chặn mà chưa đánh giá traffic thực tế, vì có thể gây false positive.

### Các rule nên xem xét

* AWS Managed Common Rule Set.
* Known Bad Inputs.
* Amazon IP Reputation List.
* Anonymous IP List.
* Rate-based rule cho request bất thường.
* Rule giới hạn truy cập các đường dẫn quản trị.

---

## 9. Áp Dụng IAM Least Privilege Cho Backend Lambda

IAM Role của Lambda backend:

```text
pharmacare-lambda-role
```

Role này được sử dụng cho các Lambda xử lý nghiệp vụ như sản phẩm, tài khoản, đơn hàng, tồn kho và database migration.

Các policy hiển thị trong cấu hình gồm:

```text
AWSLambdaBasicExecutionRole
AWSLambdaVPCAccessExecutionRole
pharmacare-cognito-admin-user-policy
pharmacare-read-rds-secret-policy
```

   ![IAM Role và policy của Backend Lambda](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs8.jpg)

### Ý nghĩa của các policy

#### `AWSLambdaBasicExecutionRole`

Cho phép Lambda ghi log vào Amazon CloudWatch Logs.

#### `AWSLambdaVPCAccessExecutionRole`

Cho phép Lambda tạo và quản lý Elastic Network Interface khi được gắn vào VPC.

#### `pharmacare-cognito-admin-user-policy`

Cấp các quyền cần thiết để backend quản lý tài khoản người dùng trong Amazon Cognito.

Policy chỉ nên cho phép các action thực sự được backend sử dụng, ví dụ:

```text
cognito-idp:AdminGetUser
cognito-idp:AdminCreateUser
cognito-idp:AdminUpdateUserAttributes
cognito-idp:AdminDisableUser
cognito-idp:AdminEnableUser
```

Resource nên giới hạn về đúng ARN của PharmaCare User Pool.

#### `pharmacare-read-rds-secret-policy`

Cho phép Lambda đọc thông tin đăng nhập database từ AWS Secrets Manager.

Action nên được giới hạn:

```text
secretsmanager:GetSecretValue
```

Resource nên giới hạn về đúng ARN của RDS Secret.

### Nguyên tắc Least Privilege

* Không sử dụng `Action: "*"` nếu không cần thiết.
* Không sử dụng `Resource: "*"` cho Cognito User Pool hoặc Secrets Manager khi đã biết ARN.
* Không dùng chung một role cho mọi chức năng nếu phạm vi quyền khác nhau đáng kể.
* Tách role backend, migration và chatbot khi cần.
* Không lưu username hoặc password database trực tiếp trong source code.
* Định kỳ kiểm tra tab **Last Accessed**.
* Xóa policy không còn sử dụng.
* Dùng IAM Access Analyzer để phát hiện quyền truy cập quá rộng.

---

## 10. Áp Dụng IAM Least Privilege Cho Lambda AI

IAM Role của hệ thống RAG:

```text
pharmacare-rag-lambda-role
```

Các policy được gắn gồm:

```text
AWSLambdaBasicExecutionRole
AWSLambdaVPCAccessExecutionRole
pharmacare-bedrock-marketplace-access-policy
pharmacare-rag-access-policy
```

   ![IAM Role và policy của Lambda AI RAG](/workshop_internship_report/images/5-Workshop/5.7-mbs/mbs9.jpg)

### Quyền cần thiết của Lambda AI

Lambda Indexing cần:

* Đọc danh sách tài liệu trong S3.
* Đọc nội dung tài liệu.
* Gọi Cohere Embed Multilingual.
* Tạo index và ghi document vào OpenSearch Serverless.
* Ghi log vào CloudWatch.
* Chạy trong VPC.

Lambda Chatbot cần:

* Gọi mô hình embedding.
* Truy vấn vector trong OpenSearch.
* Gọi Amazon Nova Micro khi `ENABLE_LLM=true`.
* Ghi log vào CloudWatch.
* Chạy trong VPC.

### Giới hạn quyền theo tài nguyên

S3:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:ListBucket",
    "s3:GetObject"
  ],
  "Resource": [
    "arn:aws:s3:::pharmacare-knowledge-docs-phu-2026",
    "arn:aws:s3:::pharmacare-knowledge-docs-phu-2026/*"
  ]
}
```

Bedrock nên giới hạn về các model thực tế được sử dụng:

```text
cohere.embed-multilingual-v3
amazon.nova-micro-v1:0
```

OpenSearch Serverless cần đồng thời có:

1. IAM Policy cho phép `aoss:APIAccessAll` trên collection cần thiết.
2. Data Access Policy cho phép đọc hoặc ghi index.

Có thể tách hai role:

```text
pharmacare-rag-indexing-role
pharmacare-rag-chatbot-role
```

Cách tách role giúp Lambda Chatbot chỉ có quyền đọc vector, trong khi Lambda Indexing mới có quyền ghi dữ liệu.

---

## 11. Kết Quả Đạt Được

Sau khi hoàn tất phần Monitoring, Backup và Security:

* Bốn Lambda function có log group riêng trong CloudWatch.
* Quản trị viên có thể theo dõi lỗi backend, migration, chatbot và indexing.
* SNS Topic `pharmacare-alert-topic` được tạo để gửi cảnh báo.
* Email subscription được cấu hình và cần xác nhận trước khi nhận thông báo.
* Bốn CloudWatch Alarm theo dõi CPU RDS, lỗi API, lỗi chatbot và lỗi backend.
* RDS được gán vào `pharmacare-rds-backup-plan`.
* Backup rule `daily-rds-backup` hỗ trợ sao lưu tự động hằng ngày.
* CloudFront được bật Core protections của AWS WAF.
* Backend Lambda và Lambda AI sử dụng IAM Role riêng.
* Các quyền truy cập được phân tách theo từng nhóm tài nguyên.
* Hệ thống có khả năng phát hiện lỗi, thông báo quản trị viên và phục hồi database.
---