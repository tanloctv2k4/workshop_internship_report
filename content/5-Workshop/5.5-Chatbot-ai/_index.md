---
title : "Building an AI RAG Chatbot with Amazon Bedrock and OpenSearch Serverless"
date : 2026-07-19
weight : 7
chapter : false
pre : " <b> 5.5 </b> "
---

## 1. Create an Amazon S3 Bucket for the Knowledge Base

Amazon S3 is used as centralized storage for documents used by the **Retrieval-Augmented Generation (RAG)** system. The documents stored in this bucket are read by the Indexing Lambda function, divided into smaller text chunks, converted into vector embeddings, and stored in Amazon OpenSearch Serverless.

### Implementation steps:

1. Open the [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1), select **Buckets**, and click **Create bucket**.
2. Configure the bucket with the following settings:

   * **AWS Region:** `Asia Pacific (Singapore) ap-southeast-1`.
   * **Bucket type:** Select **General purpose**.
   * **Bucket namespace:** Select **Global namespace**.
   * **Bucket name:** `pharmacare-knowledge-docs-phu-2026`.
   * **Object Ownership:** Select **ACLs disabled (recommended)**.
   * **Block Public Access:** Enable **Block all public access**.
   * **Bucket Versioning:** It can remain **Disabled** during the development stage.
   * **Default encryption:** Use server-side encryption with **SSE-S3**.

3. Review all configuration settings and click **Create bucket**.

   ![Create an S3 bucket for the knowledge base](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai1.jpg)

Enabling **Block all public access** prevents medical documents from being accessed directly from the Internet. The AI Lambda functions read these documents through an IAM role and an S3 Gateway Endpoint inside the VPC.

---

## 2. Create the Knowledge Base Folder Structure and Upload Documents to S3

### Implementation steps:

1. Open the bucket `pharmacare-knowledge-docs-phu-2026`.
2. Create the root folder:

   ```text
   knowledge/
   ```

3. Inside the `knowledge/` folder, organize the documents into content categories:

   ```text
   knowledge/
   ├── faq/
   ├── health-articles/
   ├── indexes/
   ├── medical-guides/
   ├── products/
   └── safety/
   ```

4. Click **Upload** and upload the text documents to the appropriate folders.
5. Check the file names, file sizes, and upload status before starting the indexing process.

   ![Knowledge base folder structure in Amazon S3](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai2.jpg)

Organizing documents by prefix makes it easier to classify data sources, track retrieved documents, and expand the knowledge base in the future. When responding to users, the chatbot can also include the file name or document category as a reference source.

---

## 3. Select AI Models in Amazon Bedrock

The system uses two types of models for different purposes:

* **Embedding model:** Converts text into numerical vectors for semantic search.
* **Large Language Model (LLM):** Synthesizes retrieved documents into natural-language responses.

### 3.1. Cohere Embed Multilingual Model

1. Open the [Amazon Bedrock Console](https://ap-southeast-1.console.aws.amazon.com/bedrock/home?region=ap-southeast-1).
2. Select **Model catalog**.
3. Search for and select **Cohere Embed Multilingual**.
4. Record the Model ID:

   ```text
   cohere.embed-multilingual-v3
   ```

5. The model creates vectors with **1024 dimensions** and supports multiple languages, making it suitable for the Vietnamese knowledge base used by PharmaCare.

   ![Cohere Embed Multilingual model in Amazon Bedrock](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai3.jpg)

The embedding model is used in both workflows:

* The Indexing Lambda function creates vectors for each document chunk.
* The Chatbot Lambda function creates a vector for the user's question.

Using the same model for both workflows allows question vectors and document vectors to be compared accurately in OpenSearch.

### 3.2. Amazon Nova Micro Model

1. In **Model catalog**, search for and select **Amazon Nova Micro**.
2. Record the Model ID:

   ```text
   amazon.nova-micro-v1:0
   ```

3. This model is used to generate natural-language responses when the `ENABLE_LLM=true` environment variable is enabled.

   ![Amazon Nova Micro model in Amazon Bedrock](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai4.jpg)

The AI processing flow is designed as follows:

```text
Document or question
        ↓
Cohere Embed Multilingual
        ↓
Vector embedding
        ↓
OpenSearch Serverless Vector Search
        ↓
Relevant documents
        ↓
Amazon Nova Micro
        ↓
Natural-language response
```

During development, Nova Micro can be disabled and the system can run in **RAG-only** mode to reduce token consumption.

---

## 4. Create Security Groups for the AI System

Because the Indexing Lambda and Chatbot Lambda functions are deployed in **Private Subnets**, the system requires dedicated Security Groups to control connections to VPC Interface Endpoints.

### 4.1. Security Group for AI Lambda Functions

1. Open the [Amazon VPC Console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1).
2. Select **Security groups** and click **Create security group**.
3. Configure the following values:

   * **Security group name:** `pharmacare-rag-lambda-sg`.
   * **Description:** `Security group for PharmaCare AI RAG Lambda functions`.
   * **VPC:** Select `pharmacare-vpc`.
   * **Inbound rules:** No inbound rule is required.
   * **Outbound rules:** Allow Lambda to send HTTPS requests to the required AWS services.

   ![Security Group for AI Lambda functions](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai5.jpg)

Lambda is the component that initiates outbound requests, so it does not need to accept inbound connections from the Internet. Leaving the inbound rule list empty reduces the attack surface.

### 4.2. Security Group for VPC Interface Endpoints

1. Create a second Security Group named:

   ```text
   pharmacare-ai-vpce-sg
   ```

2. Configure the inbound rule:

   * **Type:** HTTPS.
   * **Protocol:** TCP.
   * **Port:** `443`.
   * **Source:** Security Group `pharmacare-rag-lambda-sg`.

3. The outbound rule can remain at its default value.

   ![Security Group for AI VPC Endpoints](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai6.jpg)

Referencing the Lambda Security Group as the source ensures that only the AI Lambda functions can connect to the Interface Endpoints over HTTPS, instead of opening the endpoint to the entire VPC CIDR range.

---

## 5. Create VPC Endpoints for AI Services

The AI Lambda functions operate in Private Subnets without using a NAT Gateway. Therefore, VPC Endpoints are used to provide private connectivity to Amazon S3, Amazon Bedrock, and Amazon OpenSearch Serverless.

### 5.1. S3 Gateway Endpoint

1. In the VPC Console, select **Endpoints** and click **Create endpoint**.
2. Select the following service:

   ```text
   com.amazonaws.ap-southeast-1.s3
   ```

3. Configure the endpoint:

   * **Endpoint type:** Gateway.
   * **VPC:** `pharmacare-vpc`.
   * **Route table:** Select `pharmacare-private-rt`.
   * **Policy:** Full access can be used during testing and later restricted to the knowledge base bucket.

4. Create the endpoint and confirm that its status changes to `Available`.

   ![S3 Gateway Endpoint for AI Lambda functions](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai7.jpg)

The S3 Gateway Endpoint allows Lambda to read documents from the bucket without sending traffic through the Internet and without incurring NAT Gateway costs.

### 5.2. Bedrock Runtime Interface Endpoint

1. Create an Interface Endpoint for the following service:

   ```text
   com.amazonaws.ap-southeast-1.bedrock-runtime
   ```

2. Configure the endpoint:

   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Select the two Private App Subnets.
   * **Security Group:** `pharmacare-ai-vpce-sg`.
   * **Private DNS:** Enable **Enable private DNS name**.

3. Confirm that the endpoint status is `Available`.

   ![Bedrock Runtime Interface Endpoint](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai8.jpg)

The Lambda functions use this endpoint to invoke the Cohere Embedding model and Amazon Nova Micro through the private AWS network.

### 5.3. OpenSearch Serverless VPC Endpoint

Create a private endpoint so that the OpenSearch Serverless Collection can only be accessed from the PharmaCare VPC.

1. Create a VPC Endpoint for OpenSearch Serverless.
2. Configure:

   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Select the two Private App Subnets.
   * **Security Group:** `pharmacare-ai-vpce-sg`.
   * **Private DNS:** Enable it.

3. Confirm that the endpoint status is `Available`.

   ![OpenSearch Serverless VPC Endpoint](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai9.jpg)

### 5.4. OpenSearch Serverless Data Interface Endpoint

Create an additional Interface Endpoint for the data plane:

```text
com.amazonaws.ap-southeast-1.aoss-data
```

This endpoint handles index creation, vector insertion, and data queries from the Lambda functions.

   ![OpenSearch Serverless Data Endpoint](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai10.jpg)

After completing the endpoint configuration, the AI network flow is as follows:

```text
AI Lambda functions in Private Subnets
        ├── S3 Gateway Endpoint → S3 Knowledge Documents
        ├── Bedrock Runtime Endpoint → Embedding Model and LLM
        └── OpenSearch Serverless Endpoint → Vector Store
```

This architecture removes the need for a NAT Gateway for connections to AWS services, helping reduce costs and limiting the risk of exposing data to the public Internet.

---

## 6. Create an IAM Role for the AI Lambda Functions

### Implementation steps:

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/home#/roles).
2. Select **Roles** and click **Create role**.
3. Select **AWS service** as the trusted entity and **Lambda** as the use case.
4. Set the role name:

   ```text
   pharmacare-rag-lambda-role
   ```

5. Attach the following two AWS Managed Policies:

   * `AWSLambdaBasicExecutionRole`.
   * `AWSLambdaVPCAccessExecutionRole`.

6. Create two additional inline policies:

   * `pharmacare-bedrock-marketplace-access-policy`.
   * `pharmacare-rag-access-policy`.

   ![IAM Role for the Indexing and Chatbot Lambda functions](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai11.jpg)

This IAM Role allows the Lambda functions to:

* Write logs to Amazon CloudWatch Logs.
* Create Elastic Network Interfaces when running inside the VPC.
* Read documents from Amazon S3.
* Invoke models in Amazon Bedrock.
* Write and query data in Amazon OpenSearch Serverless.

### 6.1. Bedrock Model Access Policy

Create a policy that allows the role to invoke Cohere Embedding and use models distributed through AWS Marketplace:

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

   ![IAM policy for accessing Amazon Bedrock models](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai12.jpg)

### 6.2. S3, Bedrock, and OpenSearch Serverless Access Policy

Create the `pharmacare-rag-access-policy` policy:

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

   ![IAM policy for accessing the knowledge base and Vector Store](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai13.jpg)

In a production environment, the `Resource` field should be restricted to the exact model and OpenSearch Collection ARNs to comply with the **Principle of Least Privilege**.

---

## 7. Create the OpenSearch Serverless Vector Store

Amazon OpenSearch Serverless is used as the Vector Store for document embeddings and semantic search.

### Implementation steps:

1. Open the [Amazon OpenSearch Service Console](https://ap-southeast-1.console.aws.amazon.com/aos/home?region=ap-southeast-1).
2. Under **Serverless**, select **Collections** and click **Create collection**.
3. Configure:

   * **Collection name:** `pharmacare-rag-vector-store`.
   * **Collection type:** `Vector search`.
   * **Description:** `Vector store for PharmaCare AI RAG chatbot`.
   * **Encryption:** Use an AWS owned key in the development environment.
   * **Network access:** Select **Private**.
   * **VPC Endpoint:** Select the OpenSearch Serverless VPC Endpoint created earlier.
   * **Data access policy:** Attach `pharmacare-rag-data-policy`.

4. Click **Create** and wait until the collection status changes to `Active`.
5. Record the OpenSearch endpoint for the Lambda configuration.

   ![PharmaCare OpenSearch Serverless Vector Store](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai14.jpg)

A private collection ensures that document vectors cannot be accessed directly from the Internet. Only principals defined in the Data Access Policy and connected through the approved VPC Endpoint can access the collection.

### 7.1. Configure the Data Access Policy

Create a Data Access Policy with the following settings:

* **Policy name:** `pharmacare-rag-data-policy`.
* **Principal:** IAM Role `pharmacare-rag-lambda-role`.
* **Resource:** Indexes in the `pharmacare-rag-vector-store` collection.
* **Permissions:**

  ```text
  aoss:CreateIndex
  aoss:DeleteIndex
  aoss:UpdateIndex
  aoss:DescribeIndex
  aoss:ReadDocument
  aoss:WriteDocument
  ```

   ![Data Access Policy for OpenSearch Serverless](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai15.jpg)

The IAM Policy and the OpenSearch Data Access Policy are two independent authorization layers. Lambda must be granted permission by both layers before it can write or query vector data.

---

## 8. Build the Indexing Lambda Function

The Indexing Lambda function converts raw documents stored in S3 into vector data stored in OpenSearch Serverless.

### Main functions:

1. List documents under the `knowledge/` prefix.
2. Read the content of each file from Amazon S3.
3. Normalize the content and remove unnecessary data.
4. Split the text into smaller units called **chunks**.
5. Invoke Cohere Embed Multilingual to create an embedding for each chunk.
6. Create the index if it does not already exist.
7. Store the content, vector, and source metadata in OpenSearch Serverless.
8. Return the number of files, the number of processed chunks, and the indexing status.

### Deployment steps:

1. Open the [AWS Lambda Console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions).
2. Create or select the following function:

   ```text
   pharmacare-rag-indexing
   ```

3. Configure:

   * **Runtime:** Node.js.
   * **Execution role:** `pharmacare-rag-lambda-role`.
   * **VPC:** `pharmacare-vpc`.
   * **Subnets:** Select the two Private App Subnets.
   * **Security Group:** `pharmacare-rag-lambda-sg`.
   * **Timeout:** Set a sufficiently high timeout to process the entire knowledge base.
   * **Memory:** Adjust based on the number of documents and chunk size.

4. Package the source code with all dependencies and deploy it to Lambda. The AWS Toolkit extension in Visual Studio Code can also be used by selecting **Deploy (Ctrl+Shift+U)**.

   ![Indexing Lambda source code](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai16.jpg)

### 8.1. Configure Environment Variables

Configure the following six environment variables:

| Key | Value |
|---|---|
| `BEDROCK_REGION` | `ap-southeast-1` |
| `EMBEDDING_MODEL_ID` | `cohere.embed-multilingual-v3` |
| `KNOWLEDGE_BUCKET` | `pharmacare-knowledge-docs-phu-2026` |
| `KNOWLEDGE_PREFIX` | `knowledge/` |
| `OPENSEARCH_ENDPOINT` | `OpenSearch Serverless collection endpoint` |
| `OPENSEARCH_INDEX` | `pharmacare-knowledge-index` |

   ![Environment variables of the Indexing Lambda function](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai17.jpg)

Bucket names, endpoints, and model IDs should not be hardcoded in the source code. Environment Variables make it possible to change configuration between development, staging, and production environments without modifying the application logic.

### 8.2. Test the Indexing Lambda Function

1. Select the **Test** tab.
2. Create an empty test event:

   ```json
   {}
   ```

3. Click **Test** to start the indexing process.
4. Monitor the result in **Execution result** and Amazon CloudWatch Logs.
5. The system test produced the following results:

   * Successfully processed `250` document files.
   * Created `1186` chunks.
   * Successfully stored the data in the OpenSearch Vector Store.

   ![Indexing Lambda test result](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai18.jpg)

This result confirms that the S3 → Lambda → Bedrock Embedding → OpenSearch Serverless workflow is functioning successfully.

---

## 9. Build the Chatbot Lambda Function

The Chatbot Lambda function receives user questions and performs the RAG workflow to retrieve relevant information from the knowledge base.

### Main functions:

1. Receive a question from the frontend or API Gateway.
2. Validate the request data.
3. Invoke Cohere Embed Multilingual to create a vector for the question.
4. Query `pharmacare-knowledge-index` using vector search.
5. Retrieve the chunks with the highest similarity scores.
6. When `ENABLE_LLM=false`, return the retrieved content in RAG-only mode.
7. When `ENABLE_LLM=true`, send the context and question to Amazon Nova Micro to generate a natural-language response.
8. Return the answer together with a list of reference sources.

### Deployment steps:

1. Create the following Lambda function:

   ```text
   pharmacare-rag-chatbot
   ```

2. Configure the function to use:

   * IAM Role `pharmacare-rag-lambda-role`.
   * VPC `pharmacare-vpc`.
   * Two Private App Subnets.
   * Security Group `pharmacare-rag-lambda-sg`.

3. Deploy the chatbot source code to Lambda.

   ![PharmaCare Chatbot Lambda function](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai19.jpg)

### 9.1. Configure Environment Variables

| Key | Value |
|---|---|
| `BEDROCK_REGION` | `ap-southeast-1` |
| `EMBEDDING_MODEL_ID` | `cohere.embed-multilingual-v3` |
| `ENABLE_LLM` | `false` |
| `LLM_MAX_TOKENS` | `300` |
| `LLM_MODEL_ID` | `apac.amazon.nova-micro-v1:0` |
| `OPENSEARCH_ENDPOINT` | OpenSearch Serverless collection endpoint |
| `OPENSEARCH_INDEX` | `pharmacare-knowledge-index` |
| `TOP_K` | `3` |

   ![Environment variables of the Chatbot Lambda function](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai20.jpg)

Important environment variables:

* `ENABLE_LLM=false`: Runs the system in RAG-only mode without invoking Nova Micro.
* `LLM_MAX_TOKENS=300`: Limits the maximum generated response length.
* `TOP_K=3`: Retrieves the three document chunks with the highest similarity scores.
* `LLM_MODEL_ID`: Uses the APAC inference profile for Nova Micro.

### 9.2. Test the Chatbot Lambda Function

Create a sample test event:

```json
{
  "question": "In which cases is paracetamol used?"
}
```

Click **Test** and verify that:

* Lambda receives the question correctly.
* Bedrock creates the embedding successfully.
* OpenSearch returns relevant document chunks.
* The response includes reference content and document sources.
* No timeout, DNS, or IAM permission errors occur.

   ![Chatbot Lambda test result](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai21.jpg)

During development, `ENABLE_LLM=false` helps reduce token quota consumption. The chatbot still performs semantic search using Cohere Embedding and OpenSearch, then returns information retrieved from the internal knowledge base.

To generate more natural responses, change the configuration to:

```text
ENABLE_LLM=true
```

The retrieved documents will then be used as context for Amazon Nova Micro. The generated answer must remain grounded in the knowledge base and must not replace professional advice from a doctor or pharmacist.

---

## 10. Integrate the Chatbot into the PharmaCare Frontend

After the Chatbot Lambda function operates successfully, integrate the chatbot interface into the React application.

### Implementation steps:

1. Open the `pharmacare-frontend` project in Visual Studio Code.
2. Create a dedicated chatbot component, for example:

   ```text
   src/components/common/Chatbot.jsx
   ```

3. Import the component into `App.jsx`:

   ```jsx
   import Chatbot from "./components/common/Chatbot.jsx";
   ```

4. Place the chatbot component at the application level so that it can appear on every page:

   ```jsx
   function App() {
     return (
       <>
         {/* PharmaCare website content */}
         <Chatbot />
       </>
     );
   }
   ```

5. In the chatbot component, send the user's question to the backend API using the `POST` method.
6. Display a loading state while Lambda creates the embedding and queries OpenSearch.
7. Display the answer and reference sources after receiving the backend response.
8. Configure the API endpoint through a Vite environment variable instead of hardcoding it in the source code:

   ```env
   VITE_CHAT_API_URL=https://your-api-id.execute-api.ap-southeast-1.amazonaws.com/chat
   ```

   ![Integrate the chatbot into the React frontend](/workshop_internship_report/images/5-Workshop/5.5-chat-ai/ai22.jpg)

To prevent CORS errors, API Gateway or the Lambda Function URL must allow the PharmaCare frontend origin and support the `POST` and `OPTIONS` methods.

---

## 11. Overall Operating Principle of the AI Chatbot

The system consists of two main workflows: **Indexing** and **Chat**.

### 11.1. Knowledge Base Indexing Workflow

```text
TXT documents in Amazon S3
        ↓
Indexing Lambda
        ↓
Read and split documents into chunks
        ↓
Amazon Bedrock - Cohere Embed Multilingual
        ↓
1024-dimensional vectors
        ↓
Amazon OpenSearch Serverless
        ↓
pharmacare-knowledge-index
```

This workflow runs when the knowledge base needs to be created or updated. Each vector is stored together with its chunk content and metadata such as the file name, S3 path, or document category.

### 11.2. Question-and-Answer Workflow

```text
User enters a question in the React Frontend
        ↓
API Gateway or Lambda Function URL
        ↓
Chatbot Lambda
        ↓
Cohere Embed Multilingual creates a question vector
        ↓
OpenSearch Serverless retrieves the TOP_K closest documents
        ↓
ENABLE_LLM=false → Return RAG-only content
        or
ENABLE_LLM=true → Nova Micro generates the response
        ↓
Frontend displays the answer and reference sources
```

### 11.3. Private Network Workflow

```text
Lambda functions in Private App Subnets
        ├── S3 Gateway Endpoint
        ├── Bedrock Runtime Interface Endpoint
        └── OpenSearch Serverless Interface Endpoint
```
---

## 12. Results Achieved

After completing the configuration, the PharmaCare AI system achieved the following results:

* The knowledge base is centrally stored and encrypted in Amazon S3.
* The Indexing Lambda successfully processed `250` files and created `1186` chunks.
* Cohere Embed Multilingual creates vectors for Vietnamese documents and user questions.
* Amazon OpenSearch Serverless stores and searches vectors within a private network.
* The Chatbot Lambda retrieves relevant content and returns reference sources.
* Amazon Nova Micro can be enabled or disabled through an Environment Variable.
* The React frontend is ready to display the chatbot window across the entire website.
---