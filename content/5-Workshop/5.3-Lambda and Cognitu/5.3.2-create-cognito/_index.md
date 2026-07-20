---
title: "Cognito Authentication Deployment"
date: 2026-07-14
weight: 4
chapter: false
pre: " <b> 5.3.2 </b> "
---

# Amazon Cognito User Pool Deployment Guide - PharmaCare System

This document provides a detailed guide on establishing a user identity management solution for the **PharmaCare** system using **Amazon Cognito**. This workflow covers the initialization of the User Pool and the implementation of role-based access control through User Groups.

---

## 1. Initializing the Amazon Cognito User Pool

### Implementation Steps:
1. Open the **Amazon Cognito Console** and select **Create user pool**.
2. Configure the User Pool settings for **PharmaCare**:
   * **User pool name:** `pharmacare-user-pool`.
   * Configure sign-in methods (Email/Password), password policies, and essential security options.
3. Once completed, the system provides critical identification parameters:
   * **User Pool ID:** `ap-southeast-1_ccKkTEGWd`.
   * **Client ID:** `5pluo9vlml4kav6ib15vghsn2e`.

   ![Cognito User Pool Overview](/images/5-Workshop/5.3-lambda/cognito/cog1.png)

**Amazon Cognito User Pool** acts as a secure "User Directory," allowing **PharmaCare** to manage user registration, login, and authentication without building a custom, complex account management system. Utilizing Cognito decouples authentication logic from the application source code, enhances scalability, and ensures compliance with modern security standards.

---

## 2. Creating User Groups

### Implementation Steps:
1. Within the management interface of the newly created User Pool, navigate to the **User management** section and select **Groups**.
2. Click **Create group** to categorize users by permissions:
   * **Admin Group:** Create a group named `Admin` with the description "PharmaCare administrator group" (Precedence: 1).
   * **Customer Group:** Create a group named `Customer` with the description "PharmaCare customer group" (Precedence: 2).

   ![Creating Admin and Customer Groups in Cognito](/images/5-Workshop/5.3-lambda/cognito/cog2.png)

Implementing **User Groups** enables **PharmaCare** to enforce **Role-Based Access Control (RBAC)** effectively.
* The **Admin** group is granted privileges to manage sensitive resources and system-wide data operations.
* The **Customer** group is strictly limited to functionalities intended for end-users.
This ensures that every user is constrained to their specific authorized scope of operations, significantly reducing the risk of misuse or unauthorized data access.