# AWS Elastic Beanstalk: In-Depth Guide

AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, Node.js, PHP, Python, Ruby, Go, and Docker on familiar servers such as Apache, Nginx, Passenger, and IIS.

---

## 1. Introduction to AWS Elastic Beanstalk

Elastic Beanstalk is a **Platform as a Service (PaaS)**. You simply upload your code, and Elastic Beanstalk automatically handles the deployment, from capacity provisioning, load balancing, and auto-scaling to application health monitoring.

### Supported Platforms
*   **Web Tier**: Java, .NET, Node.js, PHP, Python, Ruby, Go, Docker.
*   **Worker Tier**: Optimized for background tasks and long-running processes (integrated with SQS).

---

## 2. Core Components

### 2.1 Application vs. Application Version
*   **Application**: A logical collection of Elastic Beanstalk components, including environments, versions, and environment configurations.
*   **Application Version**: A specific, labeled iteration of deployable code (stored in S3).

### 2.2 Environment Tier
*   **Web Server Tier**: Handles HTTP requests (standard web app). Includes an Elastic Load Balancer (ELB) and Auto Scaling Group (ASG).
*   **Worker Tier**: Pulls messages from an Amazon SQS queue to perform background tasks.

### 2.3 Configuration Template
A set of parameters and settings used to define how an environment and its resources behave.

---

## 3. Deployment Strategies (Critical)

How you deploy updates impacts your application's availability and cost.

| Strategy | Downtime | Reduced Capacity | Rollback | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **All at once** | Yes | Yes | Manual | Dev/Test (Fastest) |
| **Rolling** | No | Yes | Manual | Cost-conscious apps |
| **Rolling with Batch** | No | No | Manual | Production stability |
| **Immutable** | No | No | Instant | High-uptime Production |
| **Blue/Green** | No | No | DNS Swap | Major version changes |
| **Traffic Splitting** | No | No | Percentage | Canary testing |

### Deployment Details:
*   **Immutable**: Creates a new Auto Scaling Group with new instances. If health checks pass, the old instances are terminated.
*   **Blue/Green**: Requires two separate environments. You swap the URLs (CNAMEs) to route traffic from the old environment to the new one.

---

## 4. Advanced Configuration

### 4.1 .ebextensions
To customize your environment, include a folder named `.ebextensions` at the root of your source bundle.
*   **Format**: YAML or JSON.
*   **Files**: Must end in `.config`.
*   **Capabilities**: Install packages, create files, run shell commands, or configure AWS resources (like an RDS database).

### 4.2 Procfile
For platforms that support multiple processes (e.g., Node.js, Python), use a `Procfile` to specify the exact command to start your application.
```text
web: node server.js
```

### 4.3 Amazon Route 53 Traffic Splitting
You can configure Elastic Beanstalk to send a small percentage of traffic to a new environment (Canary testing) before fully switching over.

---

## 5. Conclusion
Elastic Beanstalk provides the "best of both worlds": the ease of a PaaS with the flexibility to "peek under the hood" and customize the underlying EC2 instances and VPC settings if needed.
