---
title: "Database Deployment"
date: 2026-07-14
weight: 2
chapter: false
pre: " <b> 5.2.2 </b> "
---

## 1. Creating the DB Subnet Group

### Implementation Steps:
1. Open the [Amazon RDS Console](https://ap-southeast-1.console.aws.amazon.com/rds/home?region=ap-southeast-1#). In the left navigation pane, select **Subnet groups** -> Click **Create DB subnet group**.
2. Configure the detailed parameters:
   * **Name:** `pharmacare-db-subnet-group`.
   * **Description:** `DB subnet group for PharmaCare AI RDS PostgreSQL`.
   * **VPC:** Select the system VPC (`pharmacare-vpc` | `vpc-00b5763254ecfdf41`).
3. In the **Add subnets** section, select 2 Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`) and specify the exact 2 Private DB Subnets (`10.0.11.0/24`, `10.0.12.0/24`). Click **Create** to finalize.

   ![DB Subnet Group Configuration](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data1.png)

 **DB Subnet Group** is a collection of subnets designated for your Amazon RDS database instances within a VPC. Selecting subnets across two different Availability Zones (Multi-AZ) ensures High Availability and robust Disaster Recovery capabilities. Furthermore, placing the database entirely within the **Private DB Subnets** layer isolates the data tier from the public internet, blocking unauthorized external access completely.

---

## 2. Provisioning the Amazon RDS PostgreSQL Database Instance

### Implementation Steps:
1. In the RDS Console, navigate to **Databases** -> Click **Create database**.
2. Choose the **Standard create** method and configure the server instance:
   * **Engine options:** Select **PostgreSQL**.
   * **DB instance identifier:** `pharmacare-db`.
   * **Instance configuration:** Choose an instance class appropriate for the system workload (e.g., `db.t4g.micro`).
   * **Connectivity:** Assign it to the `pharmacare-vpc`, select the `pharmacare-db-subnet-group` DB subnet group, and attach the dedicated RDS security group (`pharmacare-rds-sg`).
3. Review the configurations and click **Create database**. Wait until the instance status transitions to `Available`.

   ![Successful Amazon RDS PostgreSQL Initialization](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data2.png)

**Amazon RDS for PostgreSQL** provides a Fully Managed Database service responsible for processing all core business data for **PharmaCare** (such as user profiles, drug catalogs, medical records, and consultation histories). Leveraging a managed service automates heavy administrative tasks including Automated Backups, software patching, and performance monitoring, allowing the engineering team to focus entirely on application logic development.

---

## 3. Managing Credentials with AWS Secrets Manager

### Implementation Steps:
1. During the RDS initialization process (or when managing credentials post-creation), the system automatically integrates with **AWS Secrets Manager** to generate a secure secret storing the master database password.
2. Open the **AWS Secrets Manager Console** and select **Secrets** to verify the newly created secret (e.g., `rds!db-b1798063-e2c6-417e-9293-b98b072f7341`).

   ![AWS Secrets Manager Storing RDS Password](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data3.png)

**AWS Secrets Manager** resolves the critical challenge of securing sensitive information. Instead of hardcoding database usernames and passwords within source code or environment variables—where they are highly vulnerable to leakage—Secrets Manager encrypts and securely stores these credentials. When backend services (such as Lambda Functions or Application Servers) need to connect to the database, they make API calls to Secrets Manager to retrieve the credentials dynamically and securely in real time.

---

## 4. Configuring the Security Group for the VPC Endpoint

### Implementation Steps:
1. Navigate to the **VPC Console** -> Select **Security Groups** -> Click **Create security group**.
2. Configure the dedicated security group for the Secrets Manager VPC Endpoint:
   * **Security group name:** `pharmacare-vpce-sg`.
   * **Description:** `Security group for PharmaCare VPC Endpoints`.
   * **VPC:** Select `pharmacare-vpc`.
3. In the **Inbound rules** section, add a rule allowing **HTTPS (Port 443)** traffic originating from the security groups of your backend applications (e.g., the Lambda Security Group `sg-03dd8ac64cbc76d6d`). Click **Create security group**.

   ![Security Group Configuration for VPC Endpoint](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data4.png)

The `pharmacare-vpce-sg` security group acts as a network-layer checkpoint for internal traffic directed toward Secrets Manager. It enforces the Least Privilege principle by strictly permitting only encrypted HTTPS traffic (Port 443) originating from authorized internal resources (like Backend servers or Lambda functions), effectively preventing unauthorized port scanning from other network segments.

---

## 5. Deploying the VPC Endpoint for Secrets Manager

### Implementation Steps:
1. In the **VPC Console**, select **Endpoints** -> Click **Create endpoint**.
2. Configure the Endpoint parameters:
   * **Name tag:** `pharmacare-secretsmanager-vpce`.
   * **Service category:** Select **AWS services**.
   * **Service name:** Search for and select `com.amazonaws.ap-southeast-1.secretsmanager` (Type: **Interface**).
   * **VPC:** Select `pharmacare-vpc`.
   * **Subnets:** Select the Private Subnets where your applications operate.
   * **Security groups:** Attach the security group created in Step 4 (`pharmacare-vpce-sg`).
3. Click **Create endpoint** and wait until the status displays as `Available`.

   ![VPC Endpoint Status for Secrets Manager](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/data/data5.png)

This is a decisive architectural security optimization. By default, Secrets Manager is a public AWS endpoint; without a VPC Endpoint, applications residing in a Private Subnet would have to route traffic through a NAT Gateway and traverse the public Internet just to retrieve the DB credentials. Provisioning an **Interface VPC Endpoint (powered by AWS PrivateLink)** establishes a private, direct connection tunnel from within the internal VPC network straight to Secrets Manager using internal IP addresses (Private DNS). This achieves:
* **Maximized Security:** Database credential payloads never leave the isolated AWS virtual private network.
* **Enhanced Speed & Cost Efficiency:** Reduces network latency and eliminates data processing costs associated with routing traffic through the NAT Gateway for the PharmaCare system.