# AWS CodeBuild: In-Depth Guide

AWS CodeBuild is a fully managed, serverless build service that compiles source code, runs tests, and produces software packages that are ready to deploy.

---

## 1. Core Concepts & Architecture

### 1.1 What is CodeBuild?
*   **Fully Managed**: You don't need to provision, manage, or scale your own build servers (like Jenkins masters/slaves).
*   **Serverless**: It scales continuously and processes multiple builds concurrently.
*   **Pay-as-you-go**: You are charged based on the duration of the build and the compute type used (per minute).

### 1.2 Technical Deep Dive: How it Works

When you start a build, CodeBuild follows a strict orchestrated flow to ensure isolation, reproducibility, and security.

#### A. Internal Container Setup (The "Sandboxing")
1.  **Orchestration**: CodePipeline or a manual trigger sends a build request to the CodeBuild service.
2.  **Environment Provisioning**: CodeBuild spins up a **fresh, ephemeral Docker container** based on your chosen image (Amazon Linux, Ubuntu, Windows, or Custom).
3.  **Isolation**: Every build runs in its own isolated container. There is **no cross-contamination** between builds, even if they belong to the same project.
4.  **Filesystem**: The container uses a specific directory structure. Your source code is usually downloaded into `/codebuild/output/src/`.

#### B. The Detailed Phase Sequence
CodeBuild executions transition through the following technical states:

| Phase | Description |
| :--- | :--- |
| **`SUBMITTED`** | The build request is received and placed in the queue. |
| **`PROVISIONING`** | The build environment (container) is being created. This includes pulling the Docker image and setting up the network. |
| **`DOWNLOAD_SOURCE`** | CodeBuild fetches your code. If use S3/CodeCommit, it's a direct pull. If GitHub, it uses the integrated connection. |
| **`INSTALL`** | Runs the `install` phase of your buildspec. Used for setting up runtimes (e.g., `nodejs: 18`). |
| **`PRE_BUILD`** | Runs commands before the main build (e.g., logging into ECR, linting). |
| **`BUILD`** | The "primary" phase where compilation, unit testing, and packaging happen. |
| **`POST_BUILD`** | Cleanup or final packaging steps. Runs even if previous phases failed (unless catastrophic). |
| **`UPLOAD_ARTIFACTS`** | The compiled files defined in the `artifacts` section are zipped and uploaded to S3. |
| **`FINALIZING`** | Logs are flushed to CloudWatch, and status reports are sent to CodePipeline/GitHub. |
| **`COMPLETED`** | The container is destroyed, and the build enters its terminal state. |

#### C. Data Movement & Artifact Handling
*   **Artifact Affinity**: CodeBuild downloads source artifacts and uploads output artifacts to the **S3 Bucket** designated by the pipeline.
*   **Permissions**: It uses its **Service Role** to interact with these S3 buckets. If the build fails at `DOWNLOAD_SOURCE`, it's often an IAM permission or S3 bucket policy issue.
*   **Environment Variables**: Variables are injected into the shell environment *before* the `install` phase begins, making them available to all scripts.

---

## 2. The `buildspec.yml` File

The `buildspec.yml` is a YAML file that tells CodeBuild what to do. It must be placed in the **root directory** of your source code.

### 2.1 File Structure
```yaml
version: 0.2

env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    DB_PASSWORD: "/production/db/password"
  secrets-manager:
    API_KEY: "production/api/keys:stripe_key"

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - echo Installing dependencies...
      - npm install
  pre_build:
    commands:
      - echo Running unit tests...
      - npm test
  build:
    commands:
      - echo Build started on `date`
      - npm run build
  post_build:
    commands:
      - echo Build completed on `date`
      - mv dist/app.zip .

artifacts:
  files:
    - app.zip
    - 'dist/**/*'
  discard-paths: yes

cache:
  paths:
    - 'node_modules/**/*'
```

### 2.2 Key Sections
*   **env**: Manage environment variables. Integration with **Systems Manager Parameter Store** and **Secrets Manager** is a security best practice.
*   **phases**: The actual sequence of commands:
    *   `install`: Setup the environment (e.g., install specific language versions).
    *   `pre_build`: Final checks before the main build (e.g., linting, signing in to Docker).
    *   `build`: The core build/compile commands.
    *   `post_build`: Final cleanup, packaging, or notification steps.
*   **artifacts**: Specifies which files CodeBuild should save and upload to S3 for the deployment stage.
*   **cache**: Speeds up subsequent builds by storing dependencies (like `node_modules`) in S3.

---

## 3. Environment & Security

### 3.1 Supported Build Environments
CodeBuild provides a variety of environments tailored to different operating systems and hardware architectures.

| Environment Type | Operating Systems | Architecture | Use Case |
| :--- | :--- | :--- | :--- |
| **`LINUX_CONTAINER`** | Amazon Linux 2, Ubuntu | x86_64 | Standard Linux builds, Docker, Java, Go, Python. |
| **`ARM_CONTAINER`** | Amazon Linux 2, Ubuntu | ARM_64 (Graviton) | Higher performance at lower cost for ARM-compatible apps. |
| **`LINUX_GPU_CONTAINER`** | Amazon Linux 2 (with NVIDIA) | x86_64 | Machine Learning, CUDA, and graphics-intensive builds. |
| **`WINDOWS_CONTAINER`** | Windows Server | x86_64 | .NET Framework (non-Core) and Windows-native builds. |

#### Managed vs. Custom Images
*   **Managed Images**: Amazon Linux 2, Ubuntu, or Windows Server containers provided and maintained by AWS. These include pre-installed runtimes like Node.js, Python, Ruby, and Java.
*   **Custom Docker Images**: You can provide your own Docker image (stored in **Amazon ECR** or Docker Hub) if you need specific tools or dependencies not present in managed images.

#### Privileged Mode
If your build process involves **building Docker images**, you must enable **Privileged Mode**. This allows the container to run a Docker daemon inside itself (Docker-in-Docker).

### 3.2 Compute Types
CodeBuild offers different compute sizes to match your workload performance requirements:
*   `BUILD_GENERAL1_SMALL`: 3 GB RAM, 2 vCPUs.
*   `BUILD_GENERAL1_MEDIUM`: 7 GB RAM, 4 vCPUs.
*   `BUILD_GENERAL1_LARGE`: 15 GB RAM, 8 vCPUs.
*   **GPU Support**: Available for machine learning or graphics-intensive workloads.

### 3.3 Security & IAM
*   **Service Role**: CodeBuild assumes an IAM role to access your AWS resources (S3, ECR, SSM). You must follow the **Principle of Least Privilege**.

### 3.3 In-Depth: Why use CodeBuild in a VPC?

By default, CodeBuild runs in a VPC managed by AWS. This environment has public internet access but **cannot access resources inside your private VPC** (like an RDS database or a private EC2 instance). 

#### 1. Use Cases for VPC Integration
*   **Private Resource Access**: Run integration tests against a private **RDS database**, **ElastiCache** cluster, or internal microservices sitting behind a private Load Balancer.
*   **Internal Package Managers**: Connect to a private **Artifactory** or **Nexus** server hosted within your private subnets.
*   **Compliance & Audit**: Routes all build traffic through your own network, allowing you to use **VPC Flow Logs** for deep security auditing and to enforce firewall rules via **Security Groups**.

#### 2. Technical Requirements
To enable VPC integration, you must provide:
*   **Subnets**: At least two subnets in different Availability Zones (for High Availability). CodeBuild creates **Elastic Network Interfaces (ENIs)** in these subnets.
*   **Security Groups**: These act as a virtual firewall for the build container. You must ensure the Security Group allows outbound traffic to the resources the build needs (e.g., port 3306 for MySQL).
*   **NAT Gateway (CRITICAL)**: CodeBuild containers inside a VPC are **not assigned a public IP address**. If your build needs to download external dependencies (e.g., `npm install`, `docker pull`), you **MUST** have a NAT Gateway or NAT Instance in a public subnet to provide internet egress.

#### 3. Best Practice: Dedicated Subnets
It is highly recommended to use **dedicated subnets** for your CodeBuild projects. This prevents IP address exhaustion and simplifies the management of Network ACLs and Security Groups specifically for build workloads.

---

## 4. Triggers & Logging

### 4.1 How builds are started
*   **AWS CodePipeline**: Triggered automatically as part of a stage.
*   **Webhooks**: Triggered by a Git `PUSH` or `PULL_REQUEST_MERGED` event from GitHub/Bitbucket.
*   **Manual**: Started via the AWS Console or CLI.

### 4.2 Monitoring
*   **CloudWatch Logs**: All command output is streamed here. Mandatory for debugging failed builds.
*   **AWS CloudTrail**: Logs every API call made to CodeBuild for auditing purposes.

---

## 5. Hands-On: Running CodeBuild Locally

Testing your `buildspec.yml` in the cloud can be slow and costly. AWS provides a **Local Agent** that allows you to run CodeBuild environments on your own machine.

### 5.1 Prerequisites
*   **Docker**: Installed and running on your local machine.
*   **Git**: To clone the local agent repository.

### 5.2 Step-by-Step Setup

**Step 1: Pull the Local Agent Image**
AWS maintains a Docker image that simulates the CodeBuild environment.
```bash
docker pull amazon/aws-codebuild-local:latest --arch amd64
```

**Step 2: Download the Helper Script**
Download the `codebuild_build.sh` script from the official AWS GitHub repository. This script simplifies the complex `docker run` command.
```bash
curl -O https://raw.githubusercontent.com/aws/aws-codebuild-docker-images/master/local_builds/codebuild_build.sh
chmod +x codebuild_build.sh
```

**Step 3: Run the Local Build**
Navigate to the root of your project (where your `buildspec.yml` is) and execute:
```bash
./codebuild_build.sh -i <image_name> -a <artifact_dir>
```
*   `-i`: The Docker image to use (e.g., `aws/codebuild/standard:7.0`).
*   `-a`: The local directory where you want the resulting artifacts to be saved.

### 5.3 Common Use Cases & Tips
*   **Environment Variables**: You can pass environment variables using the `-e` flag in the script.
*   **Debugging**: Since it's running locally in Docker, you can inspect the container or logs instantly if a command fails in your buildspec.
*   **Speed**: Caching works differently locally; frequent builds are faster because Docker layers are cached on your disk.

---

> [!TIP]
> Use **Local Caching** if you are building Docker images to speed up the `docker build` process by reusing previous layers.
