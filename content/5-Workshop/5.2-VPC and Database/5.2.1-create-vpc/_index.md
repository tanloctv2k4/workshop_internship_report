---
title: "Create VPC"
date: 2026-07-14
weight: 1
chapter: false
pre: " <b> 5.2.1 </b> "
---
## 1. Virtual Private Cloud (VPC) Initialization

### Implementation Steps:
1. Open the [Amazon VPC console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1#Home).
2. In the navigation bar, select **Your VPCs**, then click **Create VPC**.
   
   ![VPC Creation Configuration](/images/5-Workshop/5.2-vpc-data/vpc/vpc1.png)
   
3. Configure the detailed parameters using the **VPC and more** model:
   * **Name tag auto-generation:** `pharmacare-vpc` (or an equivalent system identifier).
   * **IPv4 CIDR block:** `10.0.0.0/16` (Provides a maximum of approximately 65,536 IP addresses).
   * **Number of Availability Zones (AZs):** Select `2` (To enhance High Availability).
   * **Number of public subnets:** `2`.
   * **Number of private subnets:** `4` (Divided between the App tier and DB tier).
4. Review the visual preview diagram and click the **Create VPC** button to let the system automatically provision the resources.

   ![Successful VPC Creation Confirmation](/images/5-Workshop/5.2-vpc-data/vpc/vpc2.png)

### System-wide Impact:
This step establishes an isolated and completely independent Virtual Private Cloud (VPC) environment within the AWS Singapore region (`ap-southeast-1`). Utilizing the "VPC and more" feature automates the linking process of core components, creating a robust, secure, and standardized network skeleton from day one to deploy PharmaCare's services.

---

## 2. Subnet Verification and Configuration

### Implementation Steps:
1. In the left navigation pane, select **Subnets**.
2. Verify the list of **6 Subnets** that were automatically generated and ensure their status displays as `Available`.

   ![List of System Subnets](/images/5-Workshop/5.2-vpc-data/vpc/vpc3.png)

### System-wide Impact:
Dividing the large network address space (`10.0.0.0/16`) into **6 smaller Subnets** (`/24`) distributed evenly across 2 Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`) equips the PharmaCare system with High Fault Tolerance. If one physical AWS data center experiences an outage, the system continues to operate seamlessly in the remaining zone. Additionally, subnet segregation enforces strict security isolation across distinct architectural tiers:
* **Public Subnets (`10.0.1.0/24`, `10.0.2.0/24`):** The entry point for public internet connections (e.g., Load Balancers, Bastion Hosts).
* **Private App Subnets (`10.0.21.0/24`, `10.0.22.0/24`):** Dedicated to hosting core backend logic and application APIs. This layer is completely hidden from the public Internet.
* **Private DB Subnets (`10.0.11.0/24`, `10.0.12.0/24`):** The deepest network layer, designed to secure databases (RDS, OpenSearch, Cache), preventing maximum risk of data leakage.

---

## 3. Route Table Verification and Association

### Implementation Steps:
1. In the navigation bar, select **Route tables**.
2. Check the private route table `pharmacare-private-rt` and ensure it has successfully linked (Explicit subnet associations) with the **4 Private Subnets**.

   ![Private Route Table Configuration](/images/5-Workshop/5.2-vpc-data/vpc/vpc4.png)

3. Next, check the public route table `pharmacare-public-rt` and ensure it is associated with the **2 Public Subnets**.

   ![Public Route Table Configuration](/images/5-Workshop/5.2-vpc-data/vpc/vpc5.png)

### System-wide Impact:
Route Tables act as the traffic control system directing data flow moving in and out of the network zones:
* **Public Route Table:** Allows network traffic from Public Subnets to route directly to the Internet Gateway, enabling direct interaction with clients or end users.
* **Private Route Table:** Only permits internal network communication (Local route) or unidirectional outbound traffic when explicitly required. This configuration tightly locks down unauthorized inbound access attempts from the public Internet directly into PharmaCare's application servers or databases.

---

## 4. Internet Gateway (IGW) Verification

### Implementation Steps:
1. In the navigation bar, select **Internet gateways**.
2. Confirm that the Internet Gateway named `pharmacare-igw` has been initialized and its state is `Attached` to the correct VPC ID of the system network.

   ![Internet Gateway Status](/images/5-Workshop/5.2-vpc-data/vpc/vpc6.png)

### System-wide Impact:
The Internet Gateway (`pharmacare-igw`) serves as the sole gateway connecting PharmaCare's internal VPC network to the external public Internet. Without an IGW, the network would remain completely isolated. This component enables routing so that public-facing services (such as web interfaces or API Gateways) can receive incoming requests from customers, while also allowing the internal system to fetch and update necessary software packages and dependencies.

---

## 5. Security Group Configuration

### Implementation Steps:
1. In the left navigation pane, scroll down to the **Security** section and select **Security Groups**.
2. Inspect the existing groups or click **Create security group** to initialize virtual firewalls for each distinct service group.

   ![List of Security Groups](/images/5-Workshop/5.2-vpc-data/vpc/vpc7.png)

### System-wide Impact:
Security Groups function as Virtual Firewalls operating directly at the software/instance level. Clear segregation into dedicated groups such as `pharmacare-lambda-sg`, `pharmacare-opensearch-sg`, and `pharmacare-rds-sg` strictly enforces the **Least Privilege** principle:
* It allows granular control over exactly which Ports and source IP addresses are authorized to send data inbound or receive responses outbound.
* **Example:** Only backend servers belonging to the App Subnet group are permitted to establish connection sessions to the Database Port (`pharmacare-rds-sg`), effectively thwarting port-scanning attempts or lateral network vulnerability exploits from unauthorized resources.