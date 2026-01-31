# Hands-On: Deploying Node.js to AWS Elastic Beanstalk

This guide walks you through the practical steps of deploying a simple Node.js application to AWS Elastic Beanstalk using the AWS Management Console.

---

## 1. Prepare your Application

Create a basic Node.js application on your local machine.

### 1.1 Create `app.js`
Create a file named `app.js` with the following content:
```javascript
const http = require('http');

// Important: Elastic Beanstalk sets the PORT environment variable
const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World from AWS Elastic Beanstalk!\n');
});

server.listen(port, () => {
  console.log(`Server running at http://localhost:${port}/`);
});
```

### 1.2 Create `package.json`
Run `npm init -y` or create a `package.json` file:
```json
{
  "name": "eb-node-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
```

---

## 2. Create the Deployment Bundle

Elastic Beanstalk requires a `.zip` file containing your source code.

> [!IMPORTANT]
> Do **NOT** include the `node_modules` folder in your zip file. Elastic Beanstalk will run `npm install` automatically during deployment.

**Steps to Zip (Linux/Mac):**
```bash
zip -r my-app.zip app.js package.json
```
*Note: Make sure you are inside the project folder, so the files are at the root of the zip file.*

---

## 3. Creating the Elastic Beanstalk Environment

An **Environment** is the actual infrastructure where your application version runs.

### 3.1 Choosing the Environment Tier
When you create an environment, you must choose between two tiers:
*   **Web Server Tier**: Optimized for standard web applications that handle HTTP/HTTPS requests from an Elastic Load Balancer.
*   **Worker Tier**: Optimized for background tasks. It includes a daemon that pulls messages from an **Amazon SQS** queue and sends them as POST requests to your application.

### 3.2 Service Roles & Permissions (Essential)
Before the environment can launch, you need two types of IAM roles:
1.  **Service Role**: Allows Elastic Beanstalk to manage AWS resources for you (ELB, ASG, etc.). Usually named `aws-elasticbeanstalk-service-role`.
2.  **EC2 Instance Profile**: Allows the EC2 instances to communicate with Elastic Beanstalk, S3, and CloudWatch. Usually named `aws-elasticbeanstalk-ec2-role`.

### 3.3 Step-by-Step Creation walkthrough

#### Step 1: Initialize Application
1.  Navigate to the **Elastic Beanstalk Console** and click **Create application**.
2.  Set the name to `MyNodeApp`.

#### Step 2: Configure Environment settings
1.  **Tier**: Select **Web server tier**.
2.  **Platform**: Select **Node.js** (latest version).
3.  **Application code**: Select **Upload your code** and provide your `my-app.zip`.

#### Step 3: Configure Service Access
1.  Select **Use an existing service role** or create a new one.
2.  Select an **EC2 instance profile**.
    > [!IMPORTANT]
    > If you don't see a profile, create one with the `AWSElasticBeanstalkWebTier` and `AWSElasticBeanstalkWorkerTier` managed policies.

#### Step 4: Networking & Database
1.  **VPC**: Select your default VPC or a custom one.
2.  **Subnets**: Select at least two subnets in different availability zones for high availability.
3.  **Public IP**: Ensure your instances have a public IP if they are in public subnets.

#### Step 5: Instance Configuration
1.  **Root volume**: General Purpose SSD (gp3).
2.  **Instance types**: Choose `t3.micro` or `t3.small` for dev environments.

#### Step 6: Updates, Monitoring, and Logging
*   **Environment properties**: Under the **Updates, monitoring, and logging** tab, you can add key-value pairs.
    *   *Example*: `DB_URL` = `mydb.com`. These are accessible in Node.js via `process.env.DB_URL`.

#### Step 7: Review and Launch
1.  Review all configuration tabs and click **Submit**.
2.  Wait for the environment status to reach **Health: OK (Green)**.

---

---

## 4. Creating a Production Environment (Multi-Environment Setup)

In real-world scenarios, you often need multiple environments (e.g., `dev`, `staging`, `prod`) within the same application to isolate changes and test before high-stakes deployments.

### Why use multiple environments in one app?
*   **Consistency**: All environments share the same application versions (code bundles). You can "promote" a version that works in `dev` to `prod` without re-uploading.
*   **Isolation**: Each environment has its own URL, configuration, and underlying resources (ASG, ELB).

### Steps to create the `Prod` environment:

1.  Navigate to your **Application: MyNodeApp** dashboard in the Elastic Beanstalk Console.
2.  Click the **Create environment** button (located at the top right of the environments list).
3.  **Tier**: Select **Web server tier**.
4.  **Environment name**: `MyNodeApp-prod`.
5.  **Platform**: Select the same **Node.js** platform used for dev.
6.  **Application code**: 
    *   Select **Existing version**.
    *   Choose the **v1** (or latest) version you uploaded previously. This ensures you are deploying the exact same code to production.
7.  **Configuration Presets**: Select **High availability** (this will automatically configure a Load Balancer and multi-AZ Auto Scaling).

### Environment Isolation & Best Practices

Once the prod environment is created, you should customize it to distinguish it from dev:

#### 1. Configuration Differences
*   **Instances**: In `MyNodeApp-prod`, use larger instance types (e.g., `t3.medium`) compared to `dev` (`t3.micro`).
*   **Scaling**: Set a minimum of 2 instances in `prod` for high availability.

#### 2. Environment Properties
Update your production-specific environment variables:
*   In the `MyNodeApp-prod` environment dashboard, go to **Configuration** -> **Updates, monitoring, and logging**.
*   Update your `DB_URL` or `API_KEY` to point to production resources.

#### 3. URL Swapping (Blue/Green)
If you want to perform a Blue/Green deployment between your environments, you can use the **Environment actions** -> **Swap environment URLs** feature to redirect traffic with zero downtime.
