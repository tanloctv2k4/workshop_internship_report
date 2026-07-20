---
title : "Initialize Lambda Backend and Integrate API Gateway"
date : 2026-07-14
weight : 5
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### Overview

In this section, you will deploy a serverless backend for **PharmaCare** using **AWS Lambda**, expose backend functions through an **Amazon API Gateway HTTP API**, protect private APIs with an **Amazon Cognito JWT Authorizer**, and verify the end-to-end flow with a local **ReactJS** application.

The main request flow is:

+ The user signs in through Amazon Cognito.
+ Cognito returns a JSON Web Token (JWT) to the ReactJS application.
+ ReactJS sends HTTP requests to Amazon API Gateway.
+ API Gateway validates the JWT for protected routes.
+ API Gateway forwards valid requests to the `pharmacare-backend-api` Lambda function.
+ Lambda retrieves database connection information from AWS Secrets Manager and queries Amazon RDS PostgreSQL.

#### Deployment objectives

After completing this section, you will be able to:

+ Create a Lambda backend and connect it to a VPC.
+ Configure Lambda to use Amazon RDS and AWS Secrets Manager.
+ Package Node.js source code and upload the deployment package to Lambda.
+ Create an Amazon API Gateway HTTP API integrated with Lambda.
+ Define public and authenticated routes.
+ Configure Cross-Origin Resource Sharing (CORS) for ReactJS.
+ Create a Cognito JWT Authorizer and attach it to protected routes.
+ Test authentication and API requests from a ReactJS frontend.

#### Prerequisites

Make sure the following resources are available:

+ A VPC and Private App Subnets for Lambda.
+ An Amazon RDS PostgreSQL database reachable from Lambda.
+ An AWS Secrets Manager secret containing the RDS credentials.
+ An IAM role named `pharmacare-lambda-role` with permissions for CloudWatch Logs, Secrets Manager, and VPC networking.
+ An Amazon Cognito User Pool, an App Client, and a Customer account for testing.
+ Node.js, npm, and Visual Studio Code on the local machine.

#### Step 1: Create the Lambda backend

Open **AWS Lambda**, select **Create function**, and create a function named:

```text
pharmacare-backend-api
```

Select the **Node.js 22.x** runtime. Attach the Lambda function to the prepared VPC, Private App Subnets, and Security Group. The Lambda Security Group must be allowed to connect to the Amazon RDS Security Group on PostgreSQL port `5432`.

![Lambda backend overview](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/01-lambda-overview.png)

#### Step 2: Configure Lambda resources

Open **Configuration → General configuration → Edit** and configure:

| Setting | Value |
|---|---:|
| Memory | `256 MB` |
| Ephemeral storage | `512 MB` |
| Timeout | `10 seconds` |
| Execution role | `pharmacare-lambda-role` |

The default three-second timeout is often too short when Lambda must initialize networking, retrieve a secret, and connect to Amazon RDS.

![Lambda memory, timeout, and IAM role configuration](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/02-general-configuration.png)

#### Step 3: Add environment variables

Open **Configuration → Environment variables → Edit** and add:

| Key | Value |
|---|---|
| `DB_NAME` | `pharmacare_ai` |
| `RDS_SECRET_ARN` | ARN of the secret containing the RDS connection information |

Do not hardcode the database username or password in the source code.

![Lambda environment variables](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/03-environment-variables.png)

#### Step 4: Create the backend project

Run the following commands in the Visual Studio Code terminal:

```powershell
cd D:\ThucTapTotNghiep
mkdir pharmacare-backend-api
cd pharmacare-backend-api
npm init -y
npm install pg @aws-sdk/client-secrets-manager
```

The `pg` package provides PostgreSQL connectivity, while `@aws-sdk/client-secrets-manager` retrieves credentials from AWS Secrets Manager.

![Node.js backend project and dependencies](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/04-backend-project-dependencies.png)

#### Step 5: Implement the Lambda handler

Create `index.mjs`. The handler should:

+ Read `AWS_REGION`, `RDS_SECRET_ARN`, and `DB_NAME` from environment variables.
+ Retrieve the RDS host, port, username, and password from Secrets Manager.
+ Initialize a PostgreSQL connection pool.
+ Parse the HTTP method and route received from API Gateway.
+ Execute queries for products, categories, stores, profiles, carts, and orders.
+ Return JSON responses with the required CORS headers.

#### Step 6: Package the source code

Create `function.zip`:

```powershell
Compress-Archive `
  -Path index.mjs,node_modules,package.json,package-lock.json `
  -DestinationPath function.zip `
  -Force
```

The files must be located at the root of the ZIP package.

![Packaging the Lambda deployment file](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/05-package-lambda-code.png)

#### Step 7: Upload the deployment package

In the Lambda **Code** tab:

1. Select **Upload from → .zip file**.
2. Upload `function.zip`.
3. Confirm that the runtime is **Node.js 22.x**.
4. Confirm that the handler is `index.handler`.
5. Select **Deploy**.

![Lambda code after deployment](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/06-upload-lambda-code.png)

#### Step 8: Create an API Gateway HTTP API

Open **Amazon API Gateway → Create API** and select **HTTP API**.

![Selecting the HTTP API type](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/07-select-http-api.png)

Choose `pharmacare-backend-api` as the Lambda integration target.

![Integrating the HTTP API with Lambda](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/08-configure-api-integration.png)

#### Step 9: Create public routes

Create the following unauthenticated routes:

| Method | Resource path | Purpose |
|---|---|---|
| `GET` | `/health` | Check backend availability |
| `GET` | `/products` | Retrieve products |
| `GET` | `/products/{id}` | Retrieve product details |
| `GET` | `/categories` | Retrieve categories |
| `GET` | `/stores` | Retrieve stores |

![Public HTTP API routes](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/09-configure-public-routes.png)

#### Step 10: Configure the stage

Use the `$default` stage and enable **Auto-deploy**.

![Default stage configuration](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/10-define-default-stage.png)

Review the integrations, routes, and stage, then create the API.

![HTTP API review page](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/11-review-create-api.png)

Verify the routes under **Develop → Routes**.

![HTTP API routes after creation](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/12-api-routes-created.png)

#### Step 11: Configure CORS

Open **Develop → CORS** and configure:

| Setting | Value |
|---|---|
| Allow origins | `http://localhost:5173`, `http://localhost:3000` |
| Allow headers | `content-type`, `authorization` |
| Allow methods | `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS` |
| Max age | `3600` |
| Allow credentials | `No` |

![CORS configuration for the local frontend](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/13-configure-cors.png)

#### Step 12: Create a Cognito JWT Authorizer

Open **Develop → Authorization → Manage authorizers → Create** and select **JWT**.

| Setting | Value |
|---|---|
| Name | `pharmacare-cognito-authorizer` |
| Identity source | `$request.header.Authorization` |
| Issuer URL | `https://cognito-idp.ap-southeast-1.amazonaws.com/<USER_POOL_ID>` |
| Audience | `<APP_CLIENT_ID>` |

Do not use a client secret in a browser-based Single Page Application.

![Creating the Cognito JWT Authorizer](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/14-create-jwt-authorizer.png)

![JWT Authorizer after creation](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/15-authorizer-created.png)

#### Step 13: Protect private routes

Attach `pharmacare-cognito-authorizer` to:

| Method | Resource path |
|---|---|
| `GET` | `/profile` |
| `GET` | `/cart` |
| `POST` | `/cart` |
| `PUT` | `/cart/{productId}` |
| `DELETE` | `/cart/{productId}` |
| `POST` | `/orders` |
| `GET` | `/orders` |
| `GET` | `/orders/{id}` |

![Attaching the authorizer to GET profile](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/16-attach-authorizer-route.png)

#### Step 14: Test Cognito sign-in

Sign in with the previously created Customer account. Cognito redirects the user back to the frontend callback URL after successful authentication.

![Cognito sign-in test](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/17-cognito-login-test.png)

#### Step 15: Create the ReactJS test application

Run:

```powershell
cd D:\ThucTapTotNghiep
npm create vite@latest pharmacare-frontend -- --template react
cd pharmacare-frontend
npm install
npm install react-oidc-context
npm run dev
```

![Creating the React application with Vite](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/18-create-react-vite.png)

Open:

```text
http://localhost:5173
```

![React development server](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/19-react-development-server.png)

#### Step 16: Build a simple test interface

Create a simple interface with Cognito sign-in and sign-out buttons, public API buttons, protected API buttons, and a JSON response area. Protected requests must include:

```http
Authorization: Bearer <access_token>
```

![React test interface code](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/20-react-test-interface-code.png)

#### Step 17: Verify the result

| Test case | Expected result |
|---|---|
| Call `GET /health` without signing in | The API returns a healthy response |
| Call `GET /products` without signing in | The API returns product data |
| Call `GET /profile` without signing in | API Gateway rejects the request |
| Sign in and call `GET /profile` | The API returns the user profile |
| Call `/cart` and `/orders` with a valid JWT | Lambda processes the request and accesses RDS |
| Send an expired token or incorrect audience | API Gateway returns an authorization error |

![End-to-end test result](/images/5-Workshop/5.4-lambda-backend/5.4.1-lambda-backend-api-setup/21-api-test-result.png)

#### Result

The completed backend contains ReactJS, Amazon Cognito, Amazon API Gateway, AWS Lambda, AWS Secrets Manager, and Amazon RDS PostgreSQL. Public APIs remain available without authentication, while profile, cart, and order APIs require a valid JWT.

#### Security note

Before publishing screenshots, redact AWS Account IDs, ARNs, API IDs, User Pool IDs, App Client IDs, real callback URLs, passwords, access keys, and user tokens.