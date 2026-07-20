---
title : "S3 Storage and CloudFront CDN Distribution"
date : 2026-07-18
weight : 6
chapter : false
pre : " <b> 5.4.2 </b> "
---

## 1. Create an Amazon S3 Bucket for Product Image Storage

### Execution Steps:
1. Open the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1). In the left navigation pane, choose **Buckets**, then click **Create bucket**.
2. Configure the detailed settings for the S3 bucket:
   * **AWS Region:** Select `Asia Pacific (Singapore) ap-southeast-1` (Ensures the lowest latency and regional alignment with PharmaCare's core VPC and RDS infrastructure).
   * **Bucket type:** Choose **General purpose** (Recommended for most standard object storage architectures).
   * **Bucket name:** `pharmacare-product-images-phu-2026` (Note: S3 bucket names must be globally unique, using only lowercase letters, numbers, and hyphens).
   * **Object Ownership:** Select **ACLs disabled (recommended)** -> The system automatically applies the **Bucket owner enforced** policy. Total ownership of all objects remains with the AWS account and is centrally governed via IAM / Bucket Policies.
   * **Block Public Access settings for this bucket:** Check **Block all public access** (Ensures maximum security by preventing unauthorized direct internet access to S3; public access will be securely routed through the CDN endpoint).
   * **Bucket Versioning:** Choose **Disable** (Optimizes storage costs for static product image assets that undergo minimal changes).
   * **Default encryption:** Select **Server-side encryption with Amazon S3 managed keys (SSE-S3)** and enable **Bucket Key: Enable** to automatically encrypt data at rest while reducing encryption API call costs.
3. Review the visual configuration summary and click **Create bucket** to complete the initialization.

   ![Create S3 Bucket for PharmaCare](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket1.jpg)

Configuring the S3 Bucket with **Block all public access** enabled is a mandatory standard in PharmaCare's Cloud Native architecture. Instead of exposing the bucket directly to the internet—which introduces DDoS vulnerabilities, unauthorized scraping risks, and high data transfer costs—the system seals the origin storage and delegates public delivery exclusively to the Content Delivery Network (CloudFront CDN) leveraging AWS automated security features.

---

## 2. Create Folder Structure and Upload Product Images to S3

### Execution Steps:
1. In the **Buckets** list, click the newly created bucket name: `pharmacare-product-images-phu-2026`.
2. On the **Objects** tab, click **Create folder** to establish a hierarchical directory structure for file management.
3. Configure folder details:
   * **Folder name:** `products` (Sets a standardized root path for all product image assets).
   * **Server-side encryption:** Select **Don't specify an encryption key** (The folder will automatically inherit the bucket's default SSE-S3 encryption settings).
   * Click **Create folder** to confirm.

   ![Create products folder in S3](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket2.jpg)

4. Once the `products/` folder appears in the objects list with an active status, click directly on the folder to navigate into its internal storage space.

   ![Objects list in bucket](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket3.jpg)

5. Click **Upload**, then select **Add files** and choose the collection of PharmaCare's AI-generated product images (from `PC-0001.png` to `PC-0016.png` along with related files, totaling 193 objects).
6. Keep the storage class at **Standard** and click **Upload** to initiate data transfer to the cloud.

   ![Upload product images into products folder](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket4.jpg)

Although Amazon S3 stores data using a flat storage schema, utilizing prefix folders such as `products/` helps logically organize namespaces, facilitates object categorization, simplifies lifecycle rule implementations, and supports precise routing configurations on the CDN as the system scales.

---

## 3. Initialize Content Delivery Network (Amazon CloudFront CDN)

### Execution Steps:
1. Open the [Amazon CloudFront console](https://console.aws.amazon.com/cloudfront/v3/home), navigate to **Distributions**, and click **Create distribution**.
2. **Step 1 - Choose a plan:** Select the **Free ($0/month)** plan designed for learning and system development, which includes built-in Always-on DDoS protection, free TLS certificates, Tiered caching, and a monthly quota of 1M requests / 100GB data transfer. Click **Next**.

   ![Select CloudFront Free Plan](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket5.jpg)

3. **Step 2 - Get started (Distribution options):**
   * **Distribution name:** `pharmacare-product-images-cdn` (System identifier for the CDN network).
   * **Description:** `CloudFront CDN for PharmaCare AI product images from S3`.
   * **Distribution type:** Choose **Single website or app** (Optimizes configuration for standalone application architectures).
   * Click **Next**.

   ![Configure basic CloudFront settings](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket6.jpg)

4. **Step 3 - Specify origin:**
   * **Origin type:** Select **Amazon S3**.
   * **Origin:** Choose the previously initialized S3 origin bucket: `pharmacare-product-images-phu-2026.s3.ap-southeast-1.amazonaws.com`.
   * **Allow private S3 bucket access to CloudFront:** Check **Allow private S3 bucket access to CloudFront - Recommended** (This feature automatically grants CloudFront access by updating the S3 Bucket Policy, ensuring that only CloudFront can fetch and serve content from the restricted bucket).
   * **Origin settings:** Select **Use recommended origin settings**.
   * **Cache settings:** Select **Use recommended cache settings tailored to serving S3 content** (Optimizes edge caching for static assets).
   * Click **Next**.

   ![Configure S3 Origin for CloudFront](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket7.jpg)

5. **Step 4 - Enable security:**
   * Under **Web Application Firewall (WAF)**, the built-in security mode (**Included security protections**) is automatically activated to protect the application against common web vulnerabilities, block malicious scanning agents, and restrict IPs from threat intelligence lists.
   * Click **Next**.

   ![Enable integrated WAF security](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket8.jpg)

6. **Step 5 - Review and create:**
   * Review all overall configuration settings. Take note of the system message: *"Because you granted CloudFront access to your origin, CloudFront can write and update S3 bucket policies that restrict access to your S3 origin to CloudFront"*.
   * Click **Create distribution** to initiate the global edge network deployment process.

   ![Review and create CloudFront Distribution](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket9.jpg)

Amazon CloudFront acts as a global content delivery network powered by hundreds of Edge Locations worldwide. When end-users access the PharmaCare frontend, product images are served directly from the nearest edge server rather than routing requests back to the origin S3 bucket in Singapore, delivering substantial architectural benefits:
* **Latency Minimization:** Accelerates page loading speeds significantly, providing a seamless user experience.
* **Cost Efficiency:** Leverages edge caching to reduce the total volume of direct read requests hitting the origin S3 bucket.
* **Multi-Layered Security & WAF:** Completely masks the physical S3 origin address while effectively mitigating network-layer (Layer 3/4) and application-layer (Layer 7) DDoS attacks.

---

## 4. Verify CloudFront Distribution Deployment Status

### Execution Steps:
1. On the detailed administration page of the newly created Distribution, monitor the deployment progress until the status transitions completely from `Deploying` to `Enabled` and displays the success message: **Successfully created new distribution**.
2. Record key network parameters required for integrating with the application database in subsequent steps:
   * **Distribution domain name:** `d2sq4q0ymcz8do.cloudfront.net` (The official public domain used to access product image resources).
   * **ARN:** `arn:aws:cloudfront::392646748514:distribution/E105N1I4TWNVYQ`.

   ![Successful CloudFront deployment status](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket10.jpg)

3. **Smoke Test:** Open a web browser and test access to an actual image path using the CDN domain format:
   `https://d2sq4q0ymcz8do.cloudfront.net/products/PC-0001.png` to verify that assets are successfully served via CloudFront without requiring public read permissions on the S3 bucket.

---

## 5. Update CDN Domain Paths in Amazon RDS Database (Via AWS Lambda)

### Execution Steps:
1. Open your Integrated Development Environment (Visual Studio Code) and navigate to the project source code directory `pharmacare-db-migration`.
2. Edit the migration handler script (`index.mjs`) to automate updating the new CloudFront domain into the `image_url` column of the `products` table within the Amazon RDS database.
   * **JavaScript update handler code (`index.mjs`):**
     ```javascript
     import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";
     import pg from "pg";
     const { Client } = pg;
     
     const REGION = process.env.AWS_REGION || "ap-southeast-1";
     const SECRET_ARN = process.env.RDS_SECRET_ARN;
     const DB_NAME = process.env.DB_NAME || "pharmacare_ai";
     const DB_HOST = process.env.DB_HOST;
     const DB_PORT = Number(process.env.DB_PORT || 5432);

     // SQL query to update image paths using the CloudFront Domain
     const migrationSQL = `
       UPDATE products
       SET image_url = 'https://d2sq4q0ymcz8do.cloudfront.net/products/' || sku || '.png',
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
       // DB connection logic and migrationSQL execution...
     };
     ```

   ![Edit migration source code in VS Code](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket11.jpg)

3. Package the source code along with its dependencies (`node_modules`) into a compressed `function.zip` archive using the Terminal (PowerShell) command:
   ```powershell
   Compress-Archive -Path index.mjs,package.json,package-lock.json,node_modules -DestinationPath function.zip -Force
   ```
4. Open the [AWS Lambda console](https://ap-southeast-1.console.aws.amazon.com/lambda/home?region=ap-southeast-1#/functions), search for, and select the serverless function responsible for database migration: `pharmacare-db-migration`.
5. Under the **Code source** interface, click **Upload from** -> **.zip file** and upload the newly packaged `function.zip` file (Alternatively, use the AWS Toolkit extension directly in VS Code by pressing **Deploy (Ctrl+Shift+U)** to publish to AWS Lambda).

   ![Update and deploy code to AWS Lambda](/images/5-Workshop/5.4-lambda-backend/5.4.2-s3-product-images/s3bucket12.jpg)

6. Execute (Test) the Lambda function to trigger the SQL migration command. Verify execution logs in Amazon CloudWatch Logs to confirm that all product records in Amazon RDS have successfully updated their `image_url` fields to point toward the CloudFront CDN endpoint.

* **Secure Automation via AWS Lambda:** Instead of connecting manually to a production database from a local machine—which introduces operational risks and potential credential exposure—utilizing AWS Lambda combined with **AWS Secrets Manager** (`RDS_SECRET_ARN`) ensures the migration takes place within an isolated network environment, fully encrypted and audited via CloudWatch.
* **Completing the Cloud Native Architecture:** Following this step, all structured data (metadata, pricing, product names) is securely stored in Amazon RDS within Private Subnets, while high-capacity unstructured assets (AI images) are delivered at blazing-fast speeds via the CloudFront edge network, completing a highly optimized, cost-effective, and secure architecture for PharmaCare.