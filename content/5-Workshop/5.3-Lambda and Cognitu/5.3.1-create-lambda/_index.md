---
title: "Lambda DB Migration Deployment"
date: 2026-07-14
weight: 3
chapter: false
pre: " <b> 5.3.1 </b> "
---

## 1. Configuring the IAM Execution Role for Lambda

### Implementation Steps:
1. Open the **IAM Console**, navigate to **Roles** -> Click **Create role**.
2. Create a new IAM Role named `pharmacare-lambda-role` and attach three appropriate permissions policies:
   * `AWSLambdaBasicExecutionRole` (AWS managed): Grants Lambda permissions to write execution logs to Amazon CloudWatch.
   * `AWSLambdaVPCAccessExecutionRole` (AWS managed): Grants Lambda permissions to create Elastic Network Interfaces (ENIs) to access resources located within Private Subnets in the VPC.
   * `pharmacare-read-rds-secret-policy` (Customer inline): Custom permission allowing Lambda to read database connection credentials from AWS Secrets Manager.

   ![IAM Role Configuration for Lambda](/images/5-Workshop/5.3-lambda/lambda/lam1.png)

Applying the **Least Privilege** security model via an IAM Role ensures that the Lambda function possesses only the exact permissions required to operate. Thanks to the VPC Access permission, the Lambda function can securely communicate with the internal RDS database within the isolated network without exposing the connection protocol to the public Internet.

---

## 2. Provisioning the Migration Lambda Function

### Implementation Steps:
1. Access the **AWS Lambda Console**, select **Functions** -> Click **Create function**.
2. Choose the **Author from scratch** option and configure the basic parameters:
   * **Function name:** `pharmacare-db-migration`.
   * **Runtime:** Select **Node.js 22.x**.
   * **Architecture:** `x86_64`.
3. Under the **Permissions** section, select **Use a custom role** and specify the IAM Role created in Step 1 (`pharmacare-lambda-role`).
4. Under the **Networking (VPC)** section, connect Lambda to the internal network infrastructure:
   * **VPC:** Select `pharmacare-vpc`.
   * **Subnets:** Select the 2 Private Subnets of the system.
   * **Security groups:** Assign the dedicated Security Group for Lambda. Click **Create function** to initialize.

   ![Lambda Migration Function Provisioning](/images/5-Workshop/5.3-lambda/lambda/lam2.png)

The `pharmacare-db-migration` Lambda function serves as an automated tool to prepare the foundational data schema for the **PharmaCare** system. Instead of configuring manually or connecting directly from a personal machine to the RDS instance (which poses security risks), Lambda runs automatically within the internal VPC network, ensuring consistency and absolute security for the deployment process.

---

## 3. Tuning Resources & Timeout Settings

### Implementation Steps:
1. On the `pharmacare-db-migration` Lambda function management interface, select the **Configuration** tab -> Choose **General configuration** -> Click **Edit**.
2. Adjust the execution resource settings:
   * **Memory:** Increase to `256 MB` (ensures sufficient memory to load libraries and handle DB connections).
   * **Timeout:** Set to `1 min 0 sec` (60 seconds). Click **Save** to apply the configuration.

   ![Timeout and Memory Configuration for Lambda](/images/5-Workshop/5.3-lambda/lambda/lam3.png)

   ![Saving General Settings Confirmation](/images/5-Workshop/5.3-lambda/lambda/lam4.png)

By default, AWS Lambda has an execution timeout of just 3 seconds. The migration process requires connecting to the RDS instance, retrieving passwords from Secrets Manager, and executing SQL statements to initialize the structure of multiple complex tables for **PharmaCare** (such as `users`, prescriptions, and product catalogs). Increasing the timeout to 1 minute completely prevents unexpected Timeout Exception errors during major system updates.

---

## 4. Configuring Environment Variables

### Implementation Steps:
1. In the **Configuration** tab, select **Environment variables** -> Click **Edit**.
2. Sequentially add Key-Value pairs identifying the database connection parameters:
   * `DB_HOST`: The RDS instance endpoint (e.g., `pharmacare-db.cpiq4y2wkyzx.ap-southeast-1.rds.amazonaws.com`).
   * `DB_NAME`: The database name (`pharmacare_ai`).
   * `DB_PORT`: The access port (`5432`).
   * `RDS_SECRET_ARN`: The ARN identifier path of the secret storing the password in AWS Secrets Manager.
3. Click **Save** to apply.

   ![Environment Variables Configuration for Lambda](/images/5-Workshop/5.3-lambda/lambda/lam5.png)

Environment variables completely decouple infrastructure configuration details from the application source code. This mechanism allows you to reuse or flexibly switch database servers (from Test/Dev environments to Production) without modifying or hardcoding parameters within the code, keeping the system compliant with security principles and clean codebase management.

---

## 5. Building, Packaging, and Updating Deployment Source Code

### Implementation Steps:
1. In the local development environment (VS Code), initialize a Node.js project (`npm init -y`) and install the required dependency libraries to connect to RDS and read from Secrets Manager (`pg`, `@aws-sdk/client-secrets-manager`). Write the migration source code inside the `index.mjs` file containing the SQL statements for table creation.
2. Use the PowerShell `Compress-Archive` command in the Terminal to bundle the source code (`index.mjs`), the `node_modules` directory, and configurations into an initial compressed file named `function.zip`.

   ![Initializing and Compressing Source Code in VS Code](/images/5-Workshop/5.3-lambda/lambda/lam6.png)

3. Return to the AWS Lambda Console, under the **Code** tab, click **Upload from** -> **.zip file**, and select the compressed `function.zip` file to proceed with the upload.

   ![ZIP File Upload Interface for Lambda Update 1](/images/5-Workshop/5.3-lambda/lambda/lam7.png)

4. After updating, verify the source code displayed directly in the built-in editor (Code source) of the Lambda Console to ensure the SQL migration command structure matches completely.

   ![Verifying Lambda Source Code After Update 1](/images/5-Workshop/5.3-lambda/lambda/lam8.png)

5. If adjustments, additional tables, or logic updates are needed, re-compress the `function.zip` file a final time in VS Code and repeat the upload action via the **Upload from .zip file** interface.

   ![Final ZIP File Upload Interface for Lambda](/images/5-Workshop/5.3-lambda/lambda/lam9.png)

6. Confirm successful update (`Successfully updated the function` notification); the latest source code is now ready to trigger actual execution against the RDS PostgreSQL database.

   ![Complete Migration Source Code After Successful Update](/images/5-Workshop/5.3-lambda/lambda/lam10.png)

The packaging process using a `.zip` file ensures that all external Node.js dependency libraries (`node_modules`) are loaded directly into Lambda's runtime environment. This equips the Lambda function with the necessary tools to automatically retrieve the decryption password from Secrets Manager and execute the SQL command blocks, initializing the complete database schema and preparing a ready foundation for **PharmaCare**'s entire microservices and artificial intelligence architecture to operate.