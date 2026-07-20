---
title : "Deploy Core VPC Infrastructure and Databases"
date : 2026-07-14
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Deploy Core VPC Infrastructure and Databases

In this section, you will deploy and verify a distributed **Multi-AZ architecture** for the **Amazon RDS relational database (Pharmacy Database)** and the **Amazon OpenSearch Service vector database (Vector Store)**.

These resources are deployed inside private subnets within `VPC-Cloud 10.0.0.0/16`.

#### Why use a Multi-AZ architecture in private subnets?

+ **High availability and fault tolerance:** The **Multi-AZ architecture** distributes database resources across two Availability Zones, `AZ-A` and `AZ-B`.

+ If an infrastructure failure occurs in one Availability Zone, the system can fail over to resources in the other Availability Zone, reducing service interruption and the risk of data loss.

+ **Database isolation and security:** Amazon RDS is deployed inside the `RDS DB Subnets`. These are private subnets that do not allow direct access from the public Internet.

+ **Vector database protection:** Amazon OpenSearch Service is deployed inside the `OpenSearch VPC Subnets`, ensuring that the vector store can only be accessed by authorized resources within the VPC.

+ **Controlled network access:** Security Groups control connections from AWS Lambda to Amazon RDS and Amazon OpenSearch Service through the required service ports.

+ **Private communication:** Backend resources communicate with the databases through private IP addresses without exposing database endpoints to the public Internet.

#### Core infrastructure architecture

![Core VPC and Database Architecture](/images/5-Workshop/5.2-vpc-data/dia1.png)