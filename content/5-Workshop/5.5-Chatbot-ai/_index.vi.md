---
title : "Xây Dựng Chatbot AI RAG Với Amazon Bedrock Và OpenSearch Serverless"
date : 2026-07-19
weight : 7
chapter : false
pre : " <b> 5.5 </b> "
---

## 1. Khởi Tạo Amazon S3 Bucket Lưu Trữ Kho Tri Thức

Amazon S3 được sử dụng làm nơi lưu trữ tập trung các tài liệu phục vụ hệ thống **Retrieval-Augmented Generation (RAG)**. Các tài liệu trong bucket sẽ được Lambda Indexing đọc, chia nhỏ thành nhiều đoạn văn bản, tạo vector embedding và lưu vào Amazon OpenSearch Serverless.

### Các bước thực hiện:

1. Mở [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1), chọn **Buckets** và nhấn **Create bucket**.
2. Cấu hình bucket với các thông số:

   * **AWS Region:** `Asia Pacific (Singapore) ap-southeast-1`.
   * **Bucket type:** Chọn **General purpose**.
   * **Bucket namespace:** Chọn **Global namespace**.
   * **Bucket name:** `pharmacare-knowledge-docs-phu-2026`.
   * **Object Ownership:** Chọn **ACLs disabled (recommended)**.
   * **Block Public Access:** Bật **Block all public access**.
   * **Bucket Versioning:** Có thể để **Disable** trong giai đoạn phát triển.
   * **Default encryption:** Sử dụng mã hóa phía máy chủ **SSE-S3**.

3. Kiểm tra lại cấu hình và nhấn **Create bucket**.

   ![Khởi tạo S3 Bucket lưu trữ kho tri thức](/images/5-Workshop/5.5-chat-ai/ai1.jpg)

Việc bật **Block all public access** giúp tài liệu y tế không bị truy cập trực tiếp từ Internet. Lambda AI sẽ đọc tài liệu bằng IAM Role và đi qua S3 Gateway Endpoint trong mạng VPC.

---

## 2. Tạo Cấu Trúc Kho Tri Thức Và Tải Tài Liệu Lên S3

### Các bước thực hiện:

1. Mở bucket `pharmacare-knowledge-docs-phu-2026`.
2. Tạo thư mục gốc:

   ```text
   knowledge/
   ```

3. Bên trong thư mục `knowledge/`, tổ chức tài liệu theo từng nhóm nội dung:

   ```text
   knowledge/
   ├── faq/
   ├── health-articles/
   ├── indexes/
   ├── medical-guides/
   ├── products/
   └── safety/
   ```

4. Nhấn **Upload** và tải các tài liệu định dạng văn bản lên đúng thư mục tương ứng.
5. Kiểm tra tên tệp, dung lượng và trạng thái upload trước khi thực hiện quá trình indexing.

   ![Cấu trúc thư mục kho tri thức trên Amazon S3](/images/5-Workshop/5.5-chat-ai/ai2.jpg)

Cách tổ chức tài liệu theo prefix giúp hệ thống dễ dàng phân loại nguồn dữ liệu, theo dõi tài liệu được truy xuất và mở rộng kho tri thức trong tương lai. Khi trả lời, chatbot có thể đính kèm tên tệp hoặc nhóm tài liệu làm nguồn tham khảo.

---

## 3. Lựa Chọn Mô Hình AI Trên Amazon Bedrock

Hệ thống sử dụng hai loại mô hình với nhiệm vụ khác nhau:

* **Embedding model:** Chuyển văn bản thành vector số để tìm kiếm ngữ nghĩa.
* **Large Language Model (LLM):** Tổng hợp các tài liệu liên quan thành câu trả lời tự nhiên.

### 3.1. Mô Hình Cohere Embed Multilingual

1. Mở [Amazon Bedrock Console](https://ap-southeast-1.console.aws.amazon.com/bedrock/home?region=ap-southeast-1).
2. Chọn **Model catalog**.
3. Tìm và chọn **Cohere Embed Multilingual**.
4. Ghi nhận Model ID:

   ```text
   cohere.embed-multilingual-v3
   ```

5. Mô hình tạo vector có kích thước **1024 dimensions** và hỗ trợ nhiều ngôn ngữ, phù hợp với kho tài liệu tiếng Việt của PharmaCare.

   ![Mô hình Cohere Embed Multilingual trên Amazon Bedrock](/images/5-Workshop/5.5-chat-ai/ai3.jpg)

Mô hình embedding được sử dụng ở cả hai luồng:

* Lambda Indexing tạo vector cho từng đoạn tài liệu.
* Lambda Chatbot tạo vector cho câu hỏi của người dùng.

Nhờ sử dụng cùng một mô hình, vector của câu hỏi và vector của tài liệu có thể được so sánh chính xác trong OpenSearch.

### 3.2. Mô Hình Amazon Nova Micro

1. Trong **Model catalog**, tìm và chọn **Amazon Nova Micro**.
2. Ghi nhận Model ID:

   ```text
   amazon.nova-micro-v1:0
   ```

3. Mô hình được dùng để tạo câu trả lời tự nhiên khi biến môi trường `ENABLE_LLM=true`.

   ![Mô hình Amazon Nova Micro trên Amazon Bedrock](/images/5-Workshop/5.5-chat-ai/ai4.jpg)

Luồng xử lý AI được thiết kế như sau:

```text
Tài liệu hoặc câu hỏi
        ↓
Cohere Embed Multilingual
        ↓
Vector embedding
        ↓
OpenSearch Serverless Vector Search
        ↓
Tài liệu liên quan
        ↓
Amazon Nova Micro
        ↓
Câu trả lời tự nhiên
```

Trong giai đoạn phát triển, có thể tắt Nova Micro và chạy chế độ **RAG-only** để giảm lượng token tiêu thụ.

---

## 4. Tạo Security Group Cho Hệ Thống AI

Do Lambda Indexing và Lambda Chatbot được đặt trong **Private Subnet**, hệ thống cần Security Group riêng để kiểm soát luồng kết nối đến các VPC Interface Endpoint.

### 4.1. Security Group Cho Lambda AI

1. Mở [Amazon VPC Console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1).
2. Chọn **Security groups** và nhấn **Create security group**.
3. Cấu hình:

   * **Security group name:** `pharmacare-rag-lambda-sg`.
   * **Description:** `Security group for PharmaCare AI RAG Lambda functions`.
   * **VPC:** Chọn `pharmacare-vpc`.
   * **Inbound rules:** Không cần mở cổng inbound.
   * **Outbound rules:** Cho phép Lambda gửi yêu cầu HTTPS đến các dịch vụ AWS cần thiết.

   ![Security Group dành cho Lambda AI](/images/5-Workshop/5.5-chat-ai/ai5.jpg)

Lambda là thành phần chủ động gửi yêu cầu nên không cần nhận kết nối từ Internet. Việc không cấu hình inbound rule giúp giảm bề mặt tấn công.

### 4.2. Security Group Cho VPC Interface Endpoint

1. Tạo Security Group thứ hai với tên:

   ```text
   pharmacare-ai-vpce-sg
   ```

2. Cấu hình inbound rule:

   * **Type:** HTTPS.
   * **Protocol:** TCP.
   * **Port:** `443`.
   * **Source:** Security Group `pharmacare-rag-lambda-sg`.

3. Outbound rule có thể giữ cấu hình mặc định.

   ![Security Group dành cho AI VPC Endpoint](/images/5-Workshop/5.5-chat-ai/ai6.jpg)

Cấu hình Security Group tham chiếu lẫn nhau giúp chỉ các Lambda AI được phép kết nối đến Interface Endpoint qua HTTPS, thay vì mở cổng cho toàn bộ dải CIDR của VPC.

---

## 5. Tạo VPC Endpoint Cho Các Dịch Vụ AI

Lambda AI hoạt động trong Private Subnet và không sử dụng NAT Gateway. Vì vậy, hệ thống sử dụng VPC Endpoint để kết nối riêng tư đến Amazon S3, Amazon Bedrock và Amazon OpenSearch Serverless.

### 5.1. S3 Gateway Endpoint

1. Trong VPC Console, chọn **Endpoints** và nhấn **Create endpoint**.
2. Chọn dịch vụ:

   ```text
   com.amazonaws.ap-southeast-1.s3
   ```

3. Cấu hình:

   * **Endpoint type:** Gateway.
   * **VPC:** `pharmacare-vpc`.
   * **Route table:** Chọn `pharmacare-private-rt`.
   * **Policy:** Có thể sử dụng Full access trong giai đoạn kiểm thử, sau đó giới hạn về bucket kho tri thức.

4. Tạo endpoint và kiểm tra trạng thái chuyển sang `Available`.

   ![S3 Gateway Endpoint cho Lambda AI](/images/5-Workshop/5.5-chat-ai/ai7.jpg)

S3 Gateway Endpoint cho phép Lambda đọc tài liệu trong bucket mà không truyền dữ liệu qua Internet và không phát sinh chi phí NAT Gateway.

### 5.2. Bedrock Runtime Interface Endpoint

1. Tạo Interface Endpoint cho dịch vụ:

   ```text
   com.amazonaws.ap-southeast-1.bedrock-runtime
   ```

2. Cấu hình:

   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Chọn hai Private App Subnet.
   * **Security Group:** `pharmacare-ai-vpce-sg`.
   * **Private DNS:** Bật **Enable private DNS name**.

3. Kiểm tra trạng thái endpoint là `Available`.

   ![Bedrock Runtime Interface Endpoint](/images/5-Workshop/5.5-chat-ai/ai8.jpg)

Endpoint này được Lambda sử dụng để gọi mô hình Cohere Embedding và Amazon Nova Micro qua mạng riêng của AWS.

### 5.3. OpenSearch Serverless VPC Endpoint

Tạo endpoint riêng tư để OpenSearch Serverless Collection chỉ có thể được truy cập từ VPC của PharmaCare.

1. Tạo VPC Endpoint cho OpenSearch Serverless.
2. Chọn:

   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Hai Private App Subnet.
   * **Security Group:** `pharmacare-ai-vpce-sg`.
   * **Private DNS:** Bật.

3. Kiểm tra endpoint ở trạng thái `Available`.

   ![OpenSearch Serverless VPC Endpoint](/images/5-Workshop/5.5-chat-ai/ai9.jpg)

### 5.4. OpenSearch Serverless Data Interface Endpoint

Tạo thêm Interface Endpoint cho data plane:

```text
com.amazonaws.ap-southeast-1.aoss-data
```

Endpoint này phục vụ các yêu cầu tạo index, ghi vector và truy vấn dữ liệu từ Lambda.

   ![OpenSearch Serverless Data Endpoint](/images/5-Workshop/5.5-chat-ai/ai10.jpg)

Sau khi hoàn tất, luồng kết nối mạng của hệ thống AI như sau:

```text
Lambda AI trong Private Subnet
        ├── S3 Gateway Endpoint → S3 Knowledge Documents
        ├── Bedrock Runtime Endpoint → Embedding Model và LLM
        └── OpenSearch Serverless Endpoint → Vector Store
```

Kiến trúc này loại bỏ nhu cầu sử dụng NAT Gateway cho các kết nối nội bộ đến dịch vụ AWS, giúp giảm chi phí và hạn chế nguy cơ lộ dữ liệu ra Internet.

---

## 6. Tạo IAM Role Cho Lambda AI

### Các bước thực hiện:

1. Mở [AWS IAM Console](https://console.aws.amazon.com/iam/home#/roles).
2. Chọn **Roles** và nhấn **Create role**.
3. Chọn trusted entity là **AWS service** và use case là **Lambda**.
4. Đặt tên role:

   ```text
   pharmacare-rag-lambda-role
   ```

5. Gắn hai AWS Managed Policy:

   * `AWSLambdaBasicExecutionRole`.
   * `AWSLambdaVPCAccessExecutionRole`.

6. Tạo thêm hai inline policy:

   * `pharmacare-bedrock-marketplace-access-policy`.
   * `pharmacare-rag-access-policy`.

   ![IAM Role dành cho Lambda Indexing và Lambda Chatbot](/images/5-Workshop/5.5-chat-ai/ai11.jpg)

IAM Role này cho phép Lambda:

* Ghi log vào Amazon CloudWatch Logs.
* Tạo Elastic Network Interface khi chạy trong VPC.
* Đọc tài liệu từ S3.
* Gọi mô hình trên Amazon Bedrock.
* Ghi và truy vấn dữ liệu trong OpenSearch Serverless.

### 6.1. Policy Truy Cập Bedrock Model

Tạo policy cho phép gọi Cohere Embedding và sử dụng model thuộc AWS Marketplace:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BedrockInvokeEmbeddingModel",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": [
        "arn:aws:bedrock:ap-southeast-1::foundation-model/cohere.embed-multilingual-v3"
      ]
    },
    {
      "Sid": "AllowBedrockMarketplaceModelAccess",
      "Effect": "Allow",
      "Action": [
        "aws-marketplace:ViewSubscriptions",
        "aws-marketplace:Subscribe"
      ],
      "Resource": "*"
    }
  ]
}
```

   ![IAM Policy truy cập mô hình Bedrock](/images/5-Workshop/5.5-chat-ai/ai12.jpg)

### 6.2. Policy Truy Cập S3, Bedrock Và OpenSearch Serverless

Tạo policy `pharmacare-rag-access-policy`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3KnowledgeDocsAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::pharmacare-knowledge-docs-phu-2026",
        "arn:aws:s3:::pharmacare-knowledge-docs-phu-2026/*"
      ]
    },
    {
      "Sid": "BedrockInvokeModelAccess",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "*"
    },
    {
      "Sid": "OpenSearchServerlessAccess",
      "Effect": "Allow",
      "Action": [
        "aoss:APIAccessAll"
      ],
      "Resource": "*"
    }
  ]
}
```

   ![IAM Policy truy cập kho tri thức và Vector Store](/images/5-Workshop/5.5-chat-ai/ai13.jpg)

Trong môi trường production, nên tiếp tục giới hạn trường `Resource` về đúng ARN của model và OpenSearch Collection để tuân thủ nguyên tắc **Least Privilege**.

---

## 7. Khởi Tạo OpenSearch Serverless Vector Store

Amazon OpenSearch Serverless được sử dụng làm Vector Store để lưu embedding của các đoạn tài liệu và thực hiện tìm kiếm ngữ nghĩa.

### Các bước thực hiện:

1. Mở [Amazon OpenSearch Service Console](https://ap-southeast-1.console.aws.amazon.com/aos/home?region=ap-southeast-1).
2. Trong phần **Serverless**, chọn **Collections** và nhấn **Create collection**.
3. Cấu hình:

   * **Collection name:** `pharmacare-rag-vector-store`.
   * **Collection type:** `Vector search`.
   * **Description:** `Vector store for PharmaCare AI RAG chatbot`.
   * **Encryption:** Sử dụng AWS owned key trong môi trường phát triển.
   * **Network access:** Chọn **Private**.
   * **VPC Endpoint:** Chọn OpenSearch Serverless VPC Endpoint đã tạo.
   * **Data access policy:** Gắn policy `pharmacare-rag-data-policy`.

4. Nhấn **Create** và chờ trạng thái collection chuyển sang `Active`.
5. Ghi nhận OpenSearch endpoint để cấu hình cho Lambda.

   ![OpenSearch Serverless Vector Store của PharmaCare](/images/5-Workshop/5.5-chat-ai/ai14.jpg)

Collection private bảo đảm vector tài liệu không thể bị truy cập trực tiếp từ Internet. Chỉ các principal được khai báo trong Data Access Policy và có đường mạng qua VPC Endpoint mới được phép thao tác dữ liệu.

### 7.1. Cấu Hình Data Access Policy

Tạo Data Access Policy với các thông tin:

* **Policy name:** `pharmacare-rag-data-policy`.
* **Principal:** IAM Role `pharmacare-rag-lambda-role`.
* **Resource:** Index thuộc collection `pharmacare-rag-vector-store`.
* **Permissions:**

  ```text
  aoss:CreateIndex
  aoss:DeleteIndex
  aoss:UpdateIndex
  aoss:DescribeIndex
  aoss:ReadDocument
  aoss:WriteDocument
  ```

   ![Data Access Policy cho OpenSearch Serverless](/images/5-Workshop/5.5-chat-ai/ai15.jpg)

IAM Policy và OpenSearch Data Access Policy là hai lớp quyền độc lập. Lambda phải được cấp quyền ở cả hai lớp thì mới có thể ghi hoặc truy vấn vector.

---

## 8. Xây Dựng Lambda Indexing

Lambda Indexing chịu trách nhiệm chuyển đổi tài liệu thô trong S3 thành dữ liệu vector trong OpenSearch Serverless.

### Chức năng chính:

1. Liệt kê các tài liệu bên trong prefix `knowledge/`.
2. Đọc nội dung từng tệp từ Amazon S3.
3. Chuẩn hóa nội dung và loại bỏ dữ liệu không cần thiết.
4. Chia văn bản thành các đoạn nhỏ gọi là **chunk**.
5. Gọi Cohere Embed Multilingual để tạo embedding cho từng chunk.
6. Tạo index nếu index chưa tồn tại.
7. Ghi nội dung, vector và metadata nguồn vào OpenSearch Serverless.
8. Trả về số lượng tệp, số chunk đã xử lý và trạng thái indexing.

### Các bước triển khai:

1. Mở [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions).
2. Tạo hoặc chọn function:

   ```text
   pharmacare-rag-indexing
   ```

3. Cấu hình:

   * **Runtime:** Node.js.
   * **Execution role:** `pharmacare-rag-lambda-role`.
   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Hai Private App Subnet.
   * **Security Group:** `pharmacare-rag-lambda-sg`.
   * **Timeout:** Tăng đủ lớn để xử lý toàn bộ kho tài liệu.
   * **Memory:** Điều chỉnh phù hợp với số lượng tài liệu và kích thước chunk.

4. Đóng gói mã nguồn cùng thư viện phụ thuộc và triển khai lên Lambda. Có thể sử dụng AWS Toolkit trong Visual Studio Code và nhấn **Deploy (Ctrl+Shift+U)**.

   ![Mã nguồn Lambda Indexing](/images/5-Workshop/5.5-chat-ai/ai16.jpg)

### 8.1. Cấu Hình Environment Variables

Cấu hình sáu biến môi trường:

| Key | Value |
|---|---|
| `BEDROCK_REGION` | `ap-southeast-1` |
| `EMBEDDING_MODEL_ID` | `cohere.embed-multilingual-v3` |
| `KNOWLEDGE_BUCKET` | `pharmacare-knowledge-docs-phu-2026` |
| `KNOWLEDGE_PREFIX` | `knowledge/` |
| `OPENSEARCH_ENDPOINT` | `Endpoint của collection OpenSearch Serverless` |
| `OPENSEARCH_INDEX` | `pharmacare-knowledge-index` |

   ![Environment Variables của Lambda Indexing](/images/5-Workshop/5.5-chat-ai/ai17.jpg)

Không nên viết cố định endpoint, bucket hoặc model ID trực tiếp trong mã nguồn. Environment Variables giúp thay đổi cấu hình giữa môi trường development, staging và production mà không cần chỉnh sửa logic chương trình.

### 8.2. Kiểm Thử Lambda Indexing

1. Chọn tab **Test**.
2. Tạo test event rỗng:

   ```json
   {}
   ```

3. Nhấn **Test** để bắt đầu quá trình indexing.
4. Theo dõi kết quả trong **Execution result** và Amazon CloudWatch Logs.
5. Kết quả kiểm thử của hệ thống:

   * Xử lý thành công `250` tệp tài liệu.
   * Tạo `1186` chunk.
   * Ghi dữ liệu thành công vào OpenSearch Vector Store.

   ![Kết quả kiểm thử Lambda Indexing](/images/5-Workshop/5.5-chat-ai/ai18.jpg)

Kết quả này xác nhận luồng S3 → Lambda → Bedrock Embedding → OpenSearch Serverless đã hoạt động thành công.

---

## 9. Xây Dựng Lambda Chatbot

Lambda Chatbot tiếp nhận câu hỏi từ người dùng và thực hiện quy trình RAG để tìm nội dung liên quan trong kho tri thức.

### Chức năng chính:

1. Nhận câu hỏi từ frontend hoặc API Gateway.
2. Kiểm tra dữ liệu đầu vào.
3. Gọi Cohere Embed Multilingual để tạo vector cho câu hỏi.
4. Truy vấn `pharmacare-knowledge-index` bằng vector search.
5. Lấy các chunk có độ tương đồng cao nhất.
6. Khi `ENABLE_LLM=false`, trả về nội dung tìm được theo chế độ RAG-only.
7. Khi `ENABLE_LLM=true`, gửi context và câu hỏi đến Amazon Nova Micro để tạo câu trả lời tự nhiên.
8. Trả về câu trả lời kèm danh sách nguồn tham khảo.

### Các bước triển khai:

1. Tạo Lambda function:

   ```text
   pharmacare-rag-chatbot
   ```

2. Cấu hình function sử dụng:

   * IAM Role `pharmacare-rag-lambda-role`.
   * VPC `pharmacare-vpc`.
   * Hai Private App Subnet.
   * Security Group `pharmacare-rag-lambda-sg`.

3. Triển khai mã nguồn chatbot lên Lambda.

   ![Lambda Chatbot của PharmaCare](/images/5-Workshop/5.5-chat-ai/ai19.jpg)

### 9.1. Cấu Hình Environment Variables

| Key | Value |
|---|---|
| `BEDROCK_REGION` | `ap-southeast-1` |
| `EMBEDDING_MODEL_ID` | `cohere.embed-multilingual-v3` |
| `ENABLE_LLM` | `false` |
| `LLM_MAX_TOKENS` | `300` |
| `LLM_MODEL_ID` | `apac.amazon.nova-micro-v1:0` |
| `OPENSEARCH_ENDPOINT` | Endpoint của collection OpenSearch Serverless |
| `OPENSEARCH_INDEX` | `pharmacare-knowledge-index` |
| `TOP_K` | `3` |

   ![Environment Variables của Lambda Chatbot](/images/5-Workshop/5.5-chat-ai/ai20.jpg)

Ý nghĩa của các biến quan trọng:

* `ENABLE_LLM=false`: Chạy chế độ RAG-only, chưa gọi Nova Micro.
* `LLM_MAX_TOKENS=300`: Giới hạn độ dài nội dung sinh ra.
* `TOP_K=3`: Lấy ba đoạn tài liệu có độ tương đồng cao nhất.
* `LLM_MODEL_ID`: Sử dụng inference profile của Nova Micro tại khu vực APAC.

### 9.2. Kiểm Thử Lambda Chatbot

Tạo một test event mẫu:

```json
{
  "question": "Thuốc paracetamol được sử dụng trong trường hợp nào?"
}
```

Sau đó nhấn **Test** và kiểm tra:

* Lambda nhận đúng câu hỏi.
* Bedrock tạo embedding thành công.
* OpenSearch trả về các đoạn tài liệu liên quan.
* Response có nội dung tham khảo và nguồn tài liệu.
* Không xuất hiện lỗi timeout, DNS hoặc thiếu quyền IAM.

   ![Kết quả kiểm thử Lambda Chatbot](/images/5-Workshop/5.5-chat-ai/ai21.jpg)

Trong giai đoạn phát triển, `ENABLE_LLM=false` giúp giảm tiêu thụ quota token. Chatbot vẫn thực hiện tìm kiếm ngữ nghĩa bằng Cohere Embedding và OpenSearch, sau đó trả về nội dung từ kho tri thức nội bộ.

Khi cần câu trả lời tự nhiên hơn, chuyển cấu hình:

```text
ENABLE_LLM=true
```

Lúc này, các tài liệu được truy xuất sẽ trở thành context cho Amazon Nova Micro. Câu trả lời phải tiếp tục giới hạn trong nội dung kho tri thức và không thay thế tư vấn chuyên môn của bác sĩ hoặc dược sĩ.

---

## 10. Tích Hợp Chatbot Vào Frontend PharmaCare

Sau khi Lambda Chatbot hoạt động ổn định, thực hiện tích hợp giao diện chatbot vào ứng dụng React.

### Các bước thực hiện:

1. Mở dự án `pharmacare-frontend` trong Visual Studio Code.
2. Tạo component chatbot riêng, ví dụ:

   ```text
   src/components/common/Chatbot.jsx
   ```

3. Import component vào `App.jsx`:

   ```jsx
   import Chatbot from "./components/common/Chatbot.jsx";
   ```

4. Đặt component chatbot tại cấp ứng dụng để nó có thể hiển thị trên mọi trang:

   ```jsx
   function App() {
     return (
       <>
         {/* Nội dung website PharmaCare */}
         <Chatbot />
       </>
     );
   }
   ```

5. Trong component chatbot, gửi câu hỏi đến API backend bằng phương thức `POST`.
6. Hiển thị trạng thái đang xử lý trong thời gian Lambda tạo embedding và truy vấn OpenSearch.
7. Hiển thị câu trả lời cùng nguồn tài liệu khi backend phản hồi.
8. Cấu hình endpoint API bằng biến môi trường của Vite thay vì viết trực tiếp trong source code:

   ```env
   VITE_CHAT_API_URL=https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/chat
   ```

   ![Tích hợp Chatbot vào frontend React](/images/5-Workshop/5.5-chat-ai/ai22.jpg)

Để tránh lỗi CORS, API Gateway hoặc Lambda Function URL phải cho phép origin của frontend PharmaCare và hỗ trợ phương thức `POST`, `OPTIONS`.

---

## 11. Nguyên Lý Hoạt Động Tổng Thể Của Chatbot AI

Hệ thống gồm hai luồng chính: **Indexing** và **Chat**.

### 11.1. Luồng Indexing Kho Tri Thức

```text
Tài liệu TXT trong Amazon S3
        ↓
Lambda Indexing
        ↓
Đọc và chia tài liệu thành các chunk
        ↓
Amazon Bedrock - Cohere Embed Multilingual
        ↓
Vector 1024 chiều
        ↓
Amazon OpenSearch Serverless
        ↓
pharmacare-knowledge-index
```

Luồng này chạy khi cần tạo mới hoặc cập nhật kho tri thức. Mỗi vector được lưu cùng nội dung chunk và metadata như tên tệp, đường dẫn S3 hoặc nhóm tài liệu.

### 11.2. Luồng Hỏi Đáp

```text
Người dùng nhập câu hỏi trên React Frontend
        ↓
API Gateway hoặc Lambda Function URL
        ↓
Lambda Chatbot
        ↓
Cohere Embed Multilingual tạo vector câu hỏi
        ↓
OpenSearch Serverless tìm TOP_K tài liệu gần nhất
        ↓
ENABLE_LLM=false → Trả nội dung RAG-only
        hoặc
ENABLE_LLM=true → Nova Micro tổng hợp câu trả lời
        ↓
Frontend hiển thị câu trả lời và nguồn tham khảo
```

### 11.3. Luồng Mạng Riêng

```text
Lambda trong Private App Subnet
        ├── S3 Gateway Endpoint
        ├── Bedrock Runtime Interface Endpoint
        └── OpenSearch Serverless Interface Endpoint
```
---

## 12. Kết Quả Đạt Được

Sau khi hoàn tất cấu hình, hệ thống PharmaCare AI đạt được các kết quả:

* Kho tri thức được lưu trữ tập trung và mã hóa trên Amazon S3.
* Lambda Indexing đã xử lý thành công `250` tệp và tạo `1186` chunk.
* Cohere Embed Multilingual tạo vector cho tài liệu và câu hỏi tiếng Việt.
* Amazon OpenSearch Serverless lưu trữ và truy vấn vector trong mạng private.
* Lambda Chatbot tìm được nội dung liên quan và trả về nguồn tham khảo.
* Amazon Nova Micro có thể được bật hoặc tắt bằng Environment Variable.
* Frontend React được chuẩn bị để tích hợp cửa sổ chatbot trên toàn bộ website.
---