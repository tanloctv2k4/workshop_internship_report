---
title : "Monitoring, Backup, and Security for the PharmaCare System"
date : 2026-07-19
weight : 9
chapter : false
pre : " <b> 5.7 </b> "
---

## 1. Monitoring, Backup, and Security Overview

After deploying the frontend, backend, and AI chatbot, the PharmaCare system requires operational controls to maintain stability, protect data, and improve security when the application is exposed to Internet users.

The following services are configured in this section:

* **Amazon CloudWatch Logs:** Collects execution logs from Lambda functions.
* **Amazon SNS:** Sends alert notifications to administrators.
* **Amazon CloudWatch Alarm:** Monitors backend errors, chatbot errors, API errors, and Amazon RDS CPU utilization.
* **AWS Backup:** Creates scheduled backups of the Amazon RDS database.
* **AWS WAF:** Protects the CloudFront Distribution against malicious web requests.
* **AWS IAM:** Applies least-privilege permissions to backend and AI Lambda functions.

Monitoring and notification flow:

```text
Lambda, API Gateway, and Amazon RDS
        ↓
CloudWatch Logs and CloudWatch Metrics
        ↓
CloudWatch Alarm
        ↓
Amazon SNS Topic
        ↓
Administrator email
```

Data protection and security flow:

```text
Internet user
        ↓
Amazon CloudFront
        ↓
AWS WAF
        ↓
PharmaCare Frontend

Amazon RDS
        ↓
AWS Backup Plan
        ↓
Backup Vault
        ↓
Recovery Point
```

---

## 2. Verify CloudWatch Log Groups

Amazon CloudWatch Logs automatically stores Lambda execution logs when the function execution role includes `AWSLambdaBasicExecutionRole`.

### Implementation steps:

1. Open the [Amazon CloudWatch Console](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1).
2. Select **Logs** → **Log management**.
3. Open the **Log groups** tab.
4. Verify the PharmaCare log groups:

   ```text
   /aws/lambda/pharmacare-backend-api
   /aws/lambda/pharmacare-db-migration
   /aws/lambda/pharmacare-rag-chatbot
   /aws/lambda/pharmacare-rag-indexing
   ```

   ![PharmaCare CloudWatch Log Groups](/images/5-Workshop/5.7-mbs/mbs1.jpg)

Each Lambda function has a separate log group. CloudWatch creates log streams for individual Lambda execution environments.

### Information to monitor

* Request start and end time.
* Lambda Request ID.
* Execution duration.
* Maximum memory used.
* Amazon RDS connection errors.
* AWS Secrets Manager access errors.
* Amazon Bedrock invocation errors.
* OpenSearch Serverless DNS and timeout errors.
* Invalid chatbot requests.
* Document indexing results.

### Configure log retention

The log groups shown in the configuration use:

```text
Retention: Never expire
```

To control storage costs, configure an appropriate retention period:

| Environment | Suggested retention |
|---|---|
| Development | 7 or 14 days |
| Testing | 14 or 30 days |
| Production | 30, 60, or 90 days |
| Audit logs | Based on organizational policy |

To change retention:

1. Select a log group.
2. Select **Actions**.
3. Select **Edit retention setting**.
4. Choose the number of days.
5. Click **Save**.

---

## 3. Create an Amazon SNS Alert Topic

Amazon Simple Notification Service is used to deliver notifications when a CloudWatch Alarm enters the `ALARM` state.

### Implementation steps:

1. Open the [Amazon SNS Console](https://ap-southeast-1.console.aws.amazon.com/sns/v3/home?region=ap-southeast-1).
2. Select **Topics**.
3. Click **Create topic**.
4. Configure:

   * **Type:** `Standard`.
   * **Name:** `pharmacare-alert-topic`.
   * **Display name:** `Pharmacare Alert`.

5. Click **Create topic**.
6. Verify that the topic was created successfully.

   ![Create the PharmaCare SNS alert topic](/images/5-Workshop/5.7-mbs/mbs2.jpg)

The SNS Topic is a centralized notification channel. Multiple CloudWatch Alarms can publish to the same topic without requiring a separate email configuration for each alarm.

---

## 4. Create an Email Subscription

After creating the topic, subscribe an administrator email address.

### Implementation steps:

1. Open:

   ```text
   pharmacare-alert-topic
   ```

2. Select **Create subscription**.
3. Configure:

   * **Protocol:** `Email`.
   * **Endpoint:** Enter the administrator email address.

4. Click **Create subscription**.
5. Verify that the subscription initially displays:

   ```text
   Pending confirmation
   ```

   ![Create an email subscription for the SNS Topic](/images/5-Workshop/5.7-mbs/mbs3.jpg)

### Confirm the subscription

SNS does not send notifications while the subscription remains in the `Pending confirmation` state.

The administrator must:

1. Open the registered email inbox.
2. Find the confirmation message from AWS Notifications.
3. Click **Confirm subscription**.
4. Return to the Amazon SNS Console.
5. Verify that the status changes to:

   ```text
   Confirmed
   ```

> Check the Spam or Promotions folder when the confirmation message is not visible.

### Test SNS delivery

After confirmation:

1. Open `pharmacare-alert-topic`.
2. Click **Publish message**.
3. Enter a test subject and message.
4. Publish the message.
5. Verify that the administrator receives the email.

---

## 5. Create CloudWatch Alarms

CloudWatch Alarms monitor AWS metrics and trigger actions when a configured threshold is exceeded.

The PharmaCare system contains four alarms:

```text
pharmacare-rds-cpu-high-alarm
pharmacare-api-5xx-alarm
pharmacare-chatbot-errors-alarm
pharmacare-backend-errors-alarm
```

   ![PharmaCare CloudWatch Alarms](/images/5-Workshop/5.7-mbs/mbs4.jpg)

### 5.1. High Amazon RDS CPU Alarm

Alarm name:

```text
pharmacare-rds-cpu-high-alarm
```

Condition:

```text
CPUUtilization > 80
for 2 datapoints
within 10 minutes
```

This alarm helps detect:

* High database CPU load.
* Inefficient SQL queries.
* Too many concurrent Lambda database connections.
* An undersized RDS instance.

When the alarm is activated, review:

* RDS CPUUtilization.
* DatabaseConnections.
* FreeableMemory.
* Read and Write Latency.
* Slow queries and long-running locks.
* Lambda concurrency.

### 5.2. API 5xx Alarm

Alarm name:

```text
pharmacare-api-5xx-alarm
```

Condition:

```text
5XXError >= 1
for 1 datapoint
within 5 minutes
```

Common causes include:

* Lambda exceptions.
* Lambda timeouts.
* Amazon RDS connection failures.
* Incorrect integration responses.
* Missing IAM permissions.
* Invalid backend response formats.

### 5.3. Chatbot Lambda Error Alarm

Alarm name:

```text
pharmacare-chatbot-errors-alarm
```

Condition:

```text
Errors >= 1
for 1 datapoint
within 5 minutes
```

The alarm monitors:

```text
pharmacare-rag-chatbot
```

Important failure scenarios include:

* Bedrock model access denied.
* OpenSearch endpoint unavailable.
* Missing index.
* Vector dimension mismatch.
* Lambda timeout.
* Invalid question payload.
* Missing environment variables.

### 5.4. Backend Lambda Error Alarm

Alarm name:

```text
pharmacare-backend-errors-alarm
```

Condition:

```text
Errors >= 1
for 1 datapoint
within 5 minutes
```

This alarm monitors backend operations for products, accounts, orders, and inventory.

### 5.5. Associate the SNS Topic

In the notification configuration:

1. Select **In alarm**.
2. Select **Select an existing SNS topic**.
3. Select:

   ```text
   pharmacare-alert-topic
   ```

4. Complete the alarm configuration.

Notification flow:

```text
CloudWatch Metric
        ↓
Alarm enters ALARM
        ↓
SNS pharmacare-alert-topic
        ↓
Administrator email
```

### Alarm states

* **OK:** The metric is within the configured threshold.
* **ALARM:** The threshold has been exceeded.
* **Insufficient data:** CloudWatch does not yet have enough data.

`Insufficient data` is expected when a Lambda function has not recently been invoked or when a metric was newly created.

---

## 6. Create an AWS Backup Plan for Amazon RDS

AWS Backup automates scheduled backups of the PharmaCare database.

### Implementation steps:

1. Open the [AWS Backup Console](https://ap-southeast-1.console.aws.amazon.com/backup/home?region=ap-southeast-1).
2. Select **Backup plans**.
3. Click **Create backup plan**.
4. Create a new plan.
5. Configure:

   * **Backup plan name:** `pharmacare-rds-backup-plan`.
   * **Backup rule name:** `daily-rds-backup`.
   * **Backup vault:** `Default`.
   * **Backup frequency:** Daily.
   * **Backup window:** Select a low-traffic period.
   * **Lifecycle:** Configure the required retention period.

6. Create the backup plan.

   ![AWS Backup Plan for Amazon RDS](/images/5-Workshop/5.7-mbs/mbs5.jpg)

The plan contains one rule:

```text
daily-rds-backup
```

The rule defines:

* The backup schedule.
* The start window.
* The completion window.
* The destination Backup Vault.
* Recovery point lifecycle settings.

### Suggested retention

| Environment | Example retention |
|---|---|
| Development | 7 days |
| Testing | 7–14 days |
| Production | 30 days or longer |
| Critical data | Based on recovery and audit requirements |

The retention period should balance restoration requirements and storage cost.

---

## 7. Assign Amazon RDS to the Backup Plan

The database must be assigned to the backup plan.

### Implementation steps:

1. Open:

   ```text
   pharmacare-rds-backup-plan
   ```

2. Select **Assign resources**.
3. Configure:

   * **Resource assignment name:** `pharmacare-rds-resource-assignment`.
   * **IAM role:** `AWSBackupDefaultServiceRole`.
   * **Resource type:** Amazon RDS.
   * **Resource:** Database `pharmacare-db`.

4. Save the resource assignment.

   ![Assign Amazon RDS to the AWS Backup Plan](/images/5-Workshop/5.7-mbs/mbs6.jpg)

The Resource ID uses the following format:

```text
arn:aws:rds:ap-southeast-1:<account-id>:db:pharmacare-db
```

### Verify backup jobs

1. Open **Jobs** → **Backup jobs**.
2. Verify that a job reaches:

   ```text
   Completed
   ```

3. Open **Vaults** → `Default`.
4. Verify the RDS recovery point.
5. Review:

   * Creation time.
   * Resource ARN.
   * Expiration date.
   * Backup size.
   * Encryption key.

### Test restoration

A backup solution is reliable only after restoration has been tested.

1. Select a recovery point.
2. Click **Restore**.
3. Restore to a new test RDS instance.
4. Do not overwrite the production database.
5. Connect to the restored database.
6. Verify tables, record counts, and order data.
7. Delete the test resources after verification to avoid unnecessary charges.

---

## 8. Enable AWS WAF for CloudFront

AWS WAF protects the website by evaluating malicious web requests before they reach the CloudFront origin.

### Implementation steps:

1. Open the [Amazon CloudFront Console](https://console.aws.amazon.com/cloudfront/v3/home).
2. Select:

   ```text
   pharmacare-frontend-distribution
   ```

3. Open the **Security** tab.
4. Expand **Web Application Firewall (WAF)**.
5. Verify that **Core protections** is:

   ```text
   Enabled
   ```

6. During the initial evaluation period, use **Monitor mode** before enforcing blocking actions.

   ![AWS WAF associated with the CloudFront Distribution](/images/5-Workshop/5.7-mbs/mbs7.jpg)

### Benefits of AWS WAF

* Detects abnormal requests.
* Reduces exposure to common web vulnerabilities.
* Blocks malicious IP addresses and request patterns.
* Supports rate limiting.
* Protects frontend paths and static content.
* Records allowed and blocked requests.

### Move from monitoring to blocking

During testing:

```text
Rule action: Count or Monitor
```

After validating that legitimate traffic is not blocked, change selected rules to:

```text
Rule action: Block
```

Avoid enabling multiple blocking rules without first reviewing real traffic, because false positives may affect legitimate users.

### Rules to consider

* AWS Managed Common Rule Set.
* Known Bad Inputs.
* Amazon IP Reputation List.
* Anonymous IP List.
* Rate-based rules.
* Custom rules for administrative paths.

---

## 9. Apply Least Privilege to the Backend Lambda Role

Backend Lambda role:

```text
pharmacare-lambda-role
```

The role is used by Lambda functions that process products, accounts, orders, inventory, and database migration.

Attached policies include:

```text
AWSLambdaBasicExecutionRole
AWSLambdaVPCAccessExecutionRole
pharmacare-cognito-admin-user-policy
pharmacare-read-rds-secret-policy
```

   ![Backend Lambda IAM Role and policies](/images/5-Workshop/5.7-mbs/mbs8.jpg)

### Policy purposes

#### `AWSLambdaBasicExecutionRole`

Allows Lambda to write logs to Amazon CloudWatch Logs.

#### `AWSLambdaVPCAccessExecutionRole`

Allows Lambda to create and manage Elastic Network Interfaces while running in a VPC.

#### `pharmacare-cognito-admin-user-policy`

Allows required backend operations in Amazon Cognito.

The policy should contain only required actions, such as:

```text
cognito-idp:AdminGetUser
cognito-idp:AdminCreateUser
cognito-idp:AdminUpdateUserAttributes
cognito-idp:AdminDisableUser
cognito-idp:AdminEnableUser
```

The resource should be restricted to the PharmaCare User Pool ARN.

#### `pharmacare-read-rds-secret-policy`

Allows Lambda to retrieve database credentials from AWS Secrets Manager.

Required action:

```text
secretsmanager:GetSecretValue
```

The resource should be restricted to the RDS secret ARN.

### Least-privilege guidelines

* Avoid `Action: "*"` when it is not required.
* Avoid `Resource: "*"` for Cognito and Secrets Manager when the ARN is known.
* Do not use one role for unrelated functions with significantly different permissions.
* Separate backend, migration, and chatbot roles where appropriate.
* Never store database credentials in source code.
* Review the **Last Accessed** tab regularly.
* Remove unused policies.
* Use IAM Access Analyzer to identify overly broad access.

---

## 10. Apply Least Privilege to the AI Lambda Role

RAG system role:

```text
pharmacare-rag-lambda-role
```

Attached policies include:

```text
AWSLambdaBasicExecutionRole
AWSLambdaVPCAccessExecutionRole
pharmacare-bedrock-marketplace-access-policy
pharmacare-rag-access-policy
```

   ![AI RAG Lambda IAM Role and policies](/images/5-Workshop/5.7-mbs/mbs9.jpg)

### Required AI Lambda permissions

The Indexing Lambda requires permission to:

* List knowledge documents in S3.
* Read document objects.
* Invoke Cohere Embed Multilingual.
* Create indexes and write documents to OpenSearch Serverless.
* Write CloudWatch Logs.
* Run inside the VPC.

The Chatbot Lambda requires permission to:

* Invoke the embedding model.
* Query vectors in OpenSearch.
* Invoke Amazon Nova Micro when `ENABLE_LLM=true`.
* Write CloudWatch Logs.
* Run inside the VPC.

### Restrict access by resource

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

Bedrock access should be restricted to the models used by the application:

```text
cohere.embed-multilingual-v3
amazon.nova-micro-v1:0
```

OpenSearch Serverless requires both:

1. An IAM Policy that allows the required API access to the collection.
2. A Data Access Policy that allows index read or write operations.

The roles can be separated further:

```text
pharmacare-rag-indexing-role
pharmacare-rag-chatbot-role
```

This separation allows the Chatbot Lambda to receive read-only vector access while only the Indexing Lambda can write documents.

---

## 11. Results Achieved

After completing the Monitoring, Backup, and Security configuration:

* Four Lambda functions have separate CloudWatch log groups.
* Administrators can review backend, migration, chatbot, and indexing errors.
* The `pharmacare-alert-topic` SNS Topic provides centralized notifications.
* The email subscription is configured and must be confirmed before notifications are delivered.
* Four CloudWatch Alarms monitor RDS CPU, API errors, chatbot errors, and backend errors.
* Amazon RDS is assigned to `pharmacare-rds-backup-plan`.
* The `daily-rds-backup` rule provides scheduled daily backups.
* AWS WAF Core protections are enabled for CloudFront.
* Backend Lambda and AI Lambda use separate IAM roles.
* Permissions are divided by service and resource.
* The system can detect failures, notify administrators, and restore database data.

---