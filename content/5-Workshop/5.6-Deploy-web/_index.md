---
title : "Deploying the PharmaCare Website with Amazon S3 and CloudFront"
date : 2026-07-19
weight : 8
chapter : false
pre : " <b> 5.6 </b> "
---

## 1. Prepare the Frontend Production Build

Before deploying the website to AWS, the React source code must be compiled into optimized static files for the production environment. For a Vite project, the build output is generated in the `dist` directory.

### Implementation steps:

1. Open the PharmaCare frontend project in Visual Studio Code.
2. Verify the production environment variables, especially the API Gateway endpoint, chatbot API endpoint, and Amazon Cognito configuration.
3. Open a terminal in the project directory and install the required dependencies:

   ```bash
   npm install
   ```

4. Run the production build command:

   ```bash
   npm run build
   ```

5. After the command completes, verify the following directory:

   ```text
   dist/
   ```

6. The `dist` directory normally contains:

   ```text
   dist/
   ├── assets/
   ├── favicon.svg
   ├── index.html
   └── vite.svg
   ```

Only the contents of the `dist` directory should be uploaded to Amazon S3. The project source code, `src` directory, and `node_modules` directory must not be uploaded to the frontend bucket.

---

## 2. Create an Amazon S3 Bucket for the Frontend

Amazon S3 is used as the origin that stores the HTML, CSS, JavaScript, and other static assets of the PharmaCare website.

### Implementation steps:

1. Open the [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-southeast-1).
2. Select **Buckets** from the left navigation panel.
3. Click **Create bucket**.
4. Configure the bucket:

   * **AWS Region:** `Asia Pacific (Singapore) ap-southeast-1`.
   * **Bucket type:** Select **General purpose**.
   * **Bucket namespace:** Select **Global namespace**.
   * **Bucket name:** `pharmacare-frontend-web-phu-2026`.
   * **Object Ownership:** Select **ACLs disabled (recommended)**.
   * **Block Public Access:** Enable **Block all public access**.
   * **Bucket Versioning:** It can remain **Disabled** in the development environment.
   * **Default encryption:** Select server-side encryption with **SSE-S3**.

5. Review the settings and click **Create bucket**.

   ![Create an S3 bucket for the PharmaCare frontend](/images/5-Workshop/5.6-deploy-web/deploy1.jpg)

After the bucket is created, open it and confirm that the **Objects** tab is empty.

   ![Frontend S3 bucket created successfully](/images/5-Workshop/5.6-deploy-web/deploy2.jpg)

The bucket remains private. Users access the website through Amazon CloudFront instead of connecting directly to S3. This design hides the origin, improves security, and avoids exposing the bucket publicly.

---

## 3. Upload the `dist` Build Files to Amazon S3

### Implementation steps:

1. Open the following bucket:

   ```text
   pharmacare-frontend-web-phu-2026
   ```

2. On the **Objects** tab, click **Upload**.
3. Select **Add files** and **Add folder**.
4. Select all contents inside the `dist` directory.

> Do not upload `dist` as a parent folder. The `index.html` file must be located directly at the root of the S3 bucket.

5. Keep the Storage Class set to **S3 Standard**.
6. Click **Upload** and wait for the process to complete.
7. Confirm that the bucket contains the main objects:

   ```text
   assets/
   favicon.svg
   index.html
   vite.svg
   ```

   ![Upload the dist build files to Amazon S3](/images/5-Workshop/5.6-deploy-web/deploy3.jpg)

The deployment can also be performed with AWS CLI:

```bash
aws s3 sync dist/ s3://pharmacare-frontend-web-phu-2026 --delete
```

The `--delete` option removes old files from S3 that no longer exist in the latest build. Verify the bucket name carefully before running the command.

---

## 4. Create an Amazon CloudFront Distribution

Amazon CloudFront distributes the website through a global network of edge locations. It retrieves content from S3, caches it, and serves it to users over HTTPS.

### Implementation steps:

1. Open the [Amazon CloudFront Console](https://console.aws.amazon.com/cloudfront/v3/home).
2. Select **Distributions** and click **Create distribution**.
3. Configure the distribution:

   * **Distribution name:** `pharmacare-frontend-distribution`.
   * **Description:** `CloudFront distribution for PharmaCare AI React frontend`.
   * **Distribution type:** Select a single website or application.
   * **Origin type:** Select **Amazon S3**.
   * **Origin:** Select `pharmacare-frontend-web-phu-2026`.
   * **Private S3 access:** Allow CloudFront to access the private bucket using the AWS-recommended mechanism.
   * **Viewer protocol policy:** Redirect HTTP to HTTPS.
   * **Allowed HTTP methods:** Select `GET` and `HEAD`.
   * **Cache policy:** Use a policy optimized for static content.
   * **Compress objects automatically:** Enable it.
   * **Default root object:** Enter `index.html`.

4. Review the configuration and click **Create distribution**.
5. Wait until the distribution status changes to **Enabled**.

   ![Create a CloudFront Distribution for the PharmaCare website](/images/5-Workshop/5.6-deploy-web/deploy4.jpg)

After deployment, CloudFront provides a public domain in the following format:

```text
https://xxxxxxxxxxxxxx.cloudfront.net
```

This domain can be used to test the website before configuring a custom domain.

### Benefits of CloudFront

* Delivers content from the edge location closest to the user.
* Provides HTTPS by default.
* Prevents direct access to the S3 origin.
* Caches HTML, CSS, JavaScript, and image files.
* Reduces latency and the number of direct requests to Amazon S3.
* Supports AWS WAF and custom domains.

---

## 5. Configure Error Pages for the React Single Page Application

React uses client-side routing. When a user directly opens a route such as `/products`, `/cart`, or `/orders`, S3 may not find a matching file and return a `403` or `404` error.

CloudFront must return `index.html` so that React Router can process the requested route.

### Implementation steps:

1. Open the `pharmacare-frontend-distribution` CloudFront Distribution.
2. Select the **Error pages** tab.
3. Click **Create custom error response**.
4. Create a response for the `403` error:

   * **HTTP error code:** `403`.
   * **Customize error response:** `Yes`.
   * **Response page path:** `/index.html`.
   * **HTTP response code:** `200`.
   * **Error caching minimum TTL:** It can be set to `0` during development.

5. Create another response for the `404` error:

   * **HTTP error code:** `404`.
   * **Response page path:** `/index.html`.
   * **HTTP response code:** `200`.

   ![Configure CloudFront Error Pages for React Router](/images/5-Workshop/5.6-deploy-web/deploy5.jpg)

After this configuration, CloudFront can return the React application for routes such as:

```text
/products
/products/PC-0001
/cart
/checkout
/orders
/admin/products
```

React Router then determines which component should be displayed in the browser.

---

## 6. Clear the CloudFront Cache with an Invalidation

CloudFront may continue serving an old build because content is cached at edge locations. After each frontend update, create an invalidation so that CloudFront retrieves the latest files from S3.

### Implementation steps:

1. Open the CloudFront Distribution.
2. Select the **Invalidations** tab.
3. Click **Create invalidation**.
4. Enter:

   ```text
   /*
   ```

5. Click **Create invalidation**.
6. Wait until the status changes from `In progress` to `Completed`.

   ![Create a CloudFront Invalidation after updating the frontend](/images/5-Workshop/5.6-deploy-web/deploy6.png)

The invalidation can also be created with AWS CLI:

```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

The update workflow is:

```text
Modify React source code
        ↓
npm run build
        ↓
Synchronize dist with S3
        ↓
Create CloudFront Invalidation
        ↓
Verify the updated website
```

---

## 7. Update Amazon Cognito Callback URLs

After the frontend receives a CloudFront domain, Amazon Cognito must be updated so that the login and logout processes redirect users to the production website.

### Implementation steps:

1. Open the [Amazon Cognito Console](https://ap-southeast-1.console.aws.amazon.com/cognito/v2/idp/user-pools?region=ap-southeast-1).
2. Select the PharmaCare User Pool.
3. Select **App clients** and open the frontend App Client.
4. Open the **Login pages** configuration.
5. Add the CloudFront domain to **Allowed callback URLs**, for example:

   ```text
   https://xxxxxxxxxxxxxx.cloudfront.net/
   ```

6. Add the same domain to **Allowed sign-out URLs**:

   ```text
   https://xxxxxxxxxxxxxx.cloudfront.net/
   ```

7. The localhost URL can remain for development:

   ```text
   http://localhost:5173/
   ```

8. Click **Save changes**.

   ![Update callback and sign-out URLs in Amazon Cognito](/images/5-Workshop/5.6-deploy-web/deploy7.png)

When a custom domain is used, add it to both the callback and sign-out URL lists. The protocol, hostname, port, and path must match exactly; otherwise, Cognito may return a `redirect_mismatch` error.

---

## 8. Register a Domain with Amazon Route 53

Amazon Route 53 is used to search for, register, and manage a domain for the PharmaCare website.

### Implementation steps:

1. Open the [Amazon Route 53 Console](https://console.aws.amazon.com/route53/).
2. Select **Registered domains**.
3. Click **Register domains**.
4. Enter the preferred domain name.
5. Review its availability and registration price.
6. If the preferred domain is unavailable, select a suitable alternative.
7. Add the selected domain to the checkout list.
8. Enter the domain registrant information.
9. Enable or verify privacy protection when supported.
10. Complete the payment and verify the registration email.

   ![Search for and register a PharmaCare domain in Route 53](/images/5-Workshop/5.6-deploy-web/deploy8.jpg)

The screenshot indicates that the searched domain may not be available. In this case, select an alternative from the suggested domains or use another name that is appropriate for the PharmaCare brand.

> Searching for a domain does not mean that it has been registered. The domain is managed by the AWS account only after payment and verification are completed and Route 53 confirms the registration.

---

## 9. Associate a Custom Domain with CloudFront

After registering the domain, request an SSL/TLS certificate and associate the domain with the CloudFront Distribution.

### 9.1. Request a Certificate in AWS Certificate Manager

1. Change the AWS Region to:

   ```text
   US East (N. Virginia) us-east-1
   ```

2. Open AWS Certificate Manager.
3. Click **Request certificate**.
4. Select **Request a public certificate**.
5. Enter the domain names, for example:

   ```text
   pharmacare.example
   www.pharmacare.example
   ```

6. Select DNS validation.
7. Create the validation records in Route 53.
8. Wait until the certificate status changes to **Issued**.

A certificate used by CloudFront must be created in the `us-east-1` Region.

### 9.2. Update the CloudFront Distribution

1. Open `pharmacare-frontend-distribution`.
2. Select **Settings** and click **Edit**.
3. Add the domain under **Alternate domain names (CNAMEs)**.
4. Select the issued ACM certificate.
5. Save the changes and wait for CloudFront to redeploy.

### 9.3. Create a Route 53 DNS Record

1. Open the Hosted Zone for the domain.
2. Click **Create record**.
3. Configure:

   * **Record type:** `A`.
   * **Alias:** Enable it.
   * **Route traffic to:** Alias to CloudFront distribution.
   * **Distribution:** Select `pharmacare-frontend-distribution`.

4. Create another record if the `www` subdomain is required.
5. Wait for DNS propagation and access the website through the custom domain.

---

## 10. Deploy a New Frontend Version

For later frontend updates, run:

```bash
npm run build
aws s3 sync dist/ s3://pharmacare-frontend-web-phu-2026 --delete
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

The overall deployment workflow is:

```text
React Source Code
        ↓
Vite Production Build
        ↓
dist/
        ↓
Private Amazon S3 Bucket
        ↓
Amazon CloudFront
        ↓
Route 53 Custom Domain
        ↓
Users access the website over HTTPS
```

Amazon Cognito, API Gateway, and Lambda continue handling authentication and backend operations. CloudFront is responsible only for distributing the static frontend.

---

## 11. Results Achieved

After the deployment was completed:

* The React frontend was built into production static files.
* The files were stored in a private S3 bucket.
* CloudFront distributed the website over HTTPS.
* React Router continued working when users directly opened nested URLs.
* Invalidation updated the frontend across CloudFront edge locations.
* Cognito redirected login and logout requests to the production domain.
* Route 53 supported domain registration and DNS management.