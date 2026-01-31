# Hands-On: Automating CI/CD with AWS CodePipeline

This guide demonstrates how to create a fully automated CI/CD pipeline that triggers a deployment to AWS Elastic Beanstalk every time you push code to AWS CodeCommit.

---

## 1. Prerequisites

Before starting this guide, you **must** have the following resources ready:

1.  **Elastic Beanstalk Application**: An active environment (e.g., `MyNodeApp-dev`) running a Node.js application.
    *   *Refer to*: [08-ElasticBeanstalk-Handson.md](file:///home/user/Skills/AWS-Certified-DevOps-Engineer/01-SDLC-Automation/08-ElasticBeanstalk-Handson.md)
2.  **CodeCommit Repository**: A repository containing your Node.js source code (including `app.js` and `package.json`).
    *   *Refer to*: [03-CodeCommit-HandsOn.md](file:///home/user/Skills/AWS-Certified-DevOps-Engineer/01-SDLC-Automation/03-CodeCommit-HandsOn.md)

---

## 2. Step-by-Step Pipeline Creation

### Step 1: Initialize Pipeline
1.  Open the **AWS CodePipeline console**.
2.  Click **Create pipeline**.
3.  **Pipeline name**: `MyNodePipeline`.
4.  **Service role**: Select **New service role** (CodePipeline will automatically create the necessary IAM permissions).

### Step 2: Add Source Stage
1.  **Source provider**: AWS CodeCommit.
2.  **Repository name**: Select your repository (e.g., `my-node-repo`).
3.  **Branch name**: `main` (or the branch you are working on).
4.  **Change detection options**: **Amazon EventBridge (recommended)**. This ensures the pipeline starts immediately upon a push.

### Step 3: Add Build Stage
For this simple Node.js application, we can skip the build stage because Elastic Beanstalk handles the `npm install` and execution for us.
1.  Click **Skip build stage**.

### Step 4: Add Deploy Stage
1.  **Deploy provider**: AWS Elastic Beanstalk.
2.  **Region**: Select the region where your EB environment is located.
3.  **Application name**: `MyNodeApp`.
4.  **Environment name**: `MyNodeApp-dev` (or your staging/dev environment).

### Step 5: Review and Create
1.  Review your stages: **Source** -> **Deploy**.
2.  Click **Create pipeline**.

---

---

---

## 4. Custom Pipeline: Connecting GitHub to Elastic Beanstalk

If your code is hosted on GitHub instead of CodeCommit, you can still use CodePipeline to automate your deployments. AWS recommends using **GitHub (Version 2)** via AWS CodeStar Connections.

### Step 1: Initialize Pipeline
1.  Open the **AWS CodePipeline console** and click **Create pipeline**.
2.  **Pipeline name**: `GitHubToBeanstalk`.
3.  **Service role**: New service role.

### Step 2: Connect to GitHub (Source Stage)
1.  **Source provider**: GitHub (Version 2).
2.  **Connection**: 
    *   Click **Connect to GitHub**.
    *   **Connection name**: `MyGitHubConnection`.
    *   Click **Connect to GitHub** again.
    *   An popup will appear. Select **Install a new app** or choose an existing connection.
    *   Follow the prompts to authorize AWS to access your GitHub repositories.
3.  **Repository name**: Select your Node.js repository from the list.
4.  **Branch name**: `main`.
5.  **Output artifact format**: CodePipeline default.

### Step 3: Skip Build and Test Stages
1.  In the **Add build stage**, click **Skip build stage** and confirm.
2.  In the **Add test stage** (if visible), click **Skip test stage**.

### Step 4: Add Deploy Stage
1.  **Deploy provider**: AWS Elastic Beanstalk.
2.  **Application name**: `MyNodeApp`.
3.  **Environment name**: `MyNodeApp-dev`.

### Step 5: Review and Create
1.  Review the flow: **Source (GitHub) -> Deploy (Elastic Beanstalk)**.
2.  Click **Create pipeline**.
3.  Any change pushed to your GitHub `main` branch will now trigger an automatic deployment to Beanstalk!

---

## 5. Editing the Pipeline: Adding a Production Stage

A mature pipeline usually has multiple deployment stages (e.g., Dev -> Prod). Follow these steps to add a Production stage to your existing pipeline.

### Step 1: Enter Edit Mode
1.  Go to the **CodePipeline dashboard** and select your pipeline (`MyNodePipeline`).
2.  Click the **Edit** button at the top of the page.

### Step 2: Add the Production Stage
1.  Scroll to the bottom of the pipeline diagram.
2.  Click **+ Add stage** below your current Deploy stage.
3.  **Stage name**: `DeployToProduction`.
4.  Click **Add stage**.

### Step 3: Add the Deployment Action
1.  In the new `DeployToProduction` stage, click **+ Add action group**.
2.  **Action name**: `DeployToProd`.
3.  **Action provider**: AWS Elastic Beanstalk.
4.  **Input artifacts**: Select `SourceArtifact` (the same code that was deployed to dev).
5.  **Application name**: Select your application (`MyNodeApp`).
6.  **Environment name**: Select your production environment (`MyNodeApp-prod`).
7.  Click **Done**.

### Step 4: (Recommended) Add Manual Approval
To prevent automatic deployments to production, add a manual approval step before the Prod stage.
1.  Click **+ Add stage** *between* the Dev Deploy and Prod Deploy stages.
2.  **Stage name**: `ManualApproval`.
3.  Click **+ Add action group**.
4.  **Action name**: `ApproveProductionRelease`.
5.  **Action provider**: Manual approval.
6.  (Optional) Provide an SNS Topic ARN to notify your team via email/Slack.
7.  Click **Done**.

### Step 5: Save Changes
1.  Scroll to the top and click **Save**.
2.  Click **Release change** to test the new multi-stage flow.

---

## 6. Critical Step: Configuring IAM Permissions for Elastic Beanstalk

By default, the service role created by CodePipeline may not have sufficient permissions to perform full management actions on Elastic Beanstalk. To ensure your "Deploy" stage succeeds, you must attach the following managed policy.

### Step 1: Find your Service Role
1.  Navigate to the **IAM Console**.
2.  In the left sidebar, click **Roles**.
3.  Search for the role created for your pipeline (e.g., `AWSCodePipelineServiceRole-us-east-1-MyNodePipeline`).

### Step 2: Attach the Managed Policy
1.  Click on the role name to view its summary.
2.  Under the **Permissions** tab, click **Add permissions** -> **Attach policies**.
3.  Search for: `AdministratorAccess-AWSElasticBeanstalk`.
4.  Check the box next to the policy name and click **Add permissions**.

### Why is this needed?
This policy allows CodePipeline to create application versions, describe environments, and initiate deployments within Elastic Beanstalk. Without it, your pipeline will likely fail at the **Deploy** stage with a "Missing Permissions" error.

---

## 7. Hands-On: Multi-Region Deployment

In a high-availability scenario, you might want to deploy your application to multiple AWS regions simultaneously (e.g., `us-east-1` and `us-west-2`). CodePipeline handles the cross-region artifact replication automatically.

### Prerequisites for Multi-Region
1.  **Elastic Beanstalk Environments**: Ensure you have a "Target" environment created in the second region (e.g., `us-west-2`).
2.  **S3 Buckets**: You must have an artifact S3 bucket in **each** region where you plan to deploy.
3.  **KMS Encryption**: Cross-region pipelines **require** an AWS KMS Customer Managed Key (CMK) to encrypt and decrypt artifacts as they move between regional buckets.

### Step 1: Create the KMS Key
1.  Navigate to the **KMS Console** in your primary region.
2.  Create a **Symmetric CMK**.
3.  Update the Key Policy to allow the CodePipeline service role and any cross-region roles to use the key.

### Step 2: Add a Cross-Region Action to your Pipeline
1.  Open your pipeline (`MyNodePipeline`) and click **Edit**.
2.  Scroll to your "Deploy" stage or create a new stage called `MultiRegionDeploy`.
3.  Click **+ Add action group**.
4.  **Action name**: `DeployToWest`.
5.  **Action provider**: AWS Elastic Beanstalk.
6.  **Region**: Change from `us-east-1` to `us-west-2`.
7.  **Input artifacts**: `SourceArtifact`.
8.  **Application name**: Select your application in the second region.
9.  **Environment name**: Select your environment in the second region.
10. Click **Done**.

### Step 3: Save and Verify
1.  Click **Save** at the top.
2.  Click **Release change**.
3.  **Under the Hood**: CodePipeline will now:
    - Zip the source in `us-east-1`.
    - Automatically replicate the zip file to the S3 bucket in `us-west-2`.
    - Trigger the deployment in `us-west-2`.

### 7.4 Technical Deep Dive: Regional Artifact Storage

In a single-region pipeline, CodePipeline uses a single S3 bucket defined by the `artifactStore` key in its JSON definition. However, for multi-region deployments, this changes to a map called `artifactStores`.

#### 1. ArtifactStore vs. ArtifactStores
*   **`artifactStore`**: Used when all stages and actions reside in the same region.
*   **`artifactStores` (Plural)**: Used when actions span multiple regions. You must provide a mapping for **every region** used in the pipeline.
    ```json
    "artifactStores": {
      "us-east-1": { "type": "S3", "location": "codepipeline-us-east-1-12345" },
      "us-west-2": { "type": "S3", "location": "codepipeline-us-west-2-67890" }
    }
    ```

#### 2. The Replication Flow
CodePipeline manages the movement of data between regions to ensure "Deployment Affinity" (deploying from a local regional bucket for speed and reliability):
1.  **Source Stage**: The source action (e.g., GitHub or S3) uploads the artifact to the bucket in the **primary region** (e.g., `us-east-1`).
2.  **Cross-Region Replication**: CodePipeline's backend service detects that a downstream action is in a different region (`us-west-2`).
3.  **Automatic Copy**: It automatically copies the artifact from the `us-east-1` bucket to the `us-west-2` bucket.
4.  **Local Execution**: The deployment action in `us-west-2` downloads the artifact from its **local regional bucket** to perform the deployment.

#### 3. Why KMS CMK is Mandatory
*   **Cross-Region Permissions**: AWS managed keys (like `aws/s3`) are region-specific and cannot be shared across regions.
*   **Decryption**: For CodePipeline to read from the primary bucket and write to the secondary bucket, it needs a **Customer Managed Key (CMK)** with a policy that explicitly allows cross-region access. Without a CMK, the artifact replication will fail with an "Access Denied" error.

---

## 8. Creating a Pipeline via AWS CLI (Programmatic Way)

For automation and Version Control of your infrastructure, you can create pipelines using the AWS CLI and a JSON definition.

### Step 1: Create a Pipeline JSON File
Create a file named `pipeline.json`. This example defines a standard pipeline that connects CodeCommit to Elastic Beanstalk.

```json
{
    "pipeline": {
        "name": "MyNodePipeline-CLI",
        "roleArn": "arn:aws:iam::123456789012:role/service-role/AWSCodePipelineServiceRole",
        "artifactStore": {
            "type": "S3",
            "location": "codepipeline-us-east-1-123456789012"
        },
        "stages": [
            {
                "name": "Source",
                "actions": [
                    {
                        "name": "SourceAction",
                        "actionTypeId": {
                            "category": "Source",
                            "owner": "AWS",
                            "provider": "CodeCommit",
                            "version": "1"
                        },
                        "configuration": {
                            "RepositoryName": "my-node-repo",
                            "BranchName": "main"
                        },
                        "outputArtifacts": [
                            {
                                "name": "SourceArtifact"
                            }
                        ],
                        "runOrder": 1
                    }
                ]
            },
            {
                "name": "Deploy",
                "actions": [
                    {
                        "name": "DeployAction",
                        "actionTypeId": {
                            "category": "Deploy",
                            "owner": "AWS",
                            "provider": "ElasticBeanstalk",
                            "version": "1"
                        },
                        "configuration": {
                            "ApplicationName": "MyNodeApp",
                            "EnvironmentName": "MyNodeApp-dev"
                        },
                        "inputArtifacts": [
                            {
                                "name": "SourceArtifact"
                            }
                        ],
                        "runOrder": 1
                    }
                ]
            }
        ],
        "version": 1
    }
}
```
> [!NOTE]
> Replace `roleArn`, `location`, and names with your actual AWS resources.

### Step 2: Run the CLI Command
Execute the following command to create the pipeline:
```bash
aws codepipeline create-pipeline --cli-input-json file://pipeline.json
```

### Step 3: Useful CLI Commands
*   **Get current status**: `aws codepipeline get-pipeline-state --name MyNodePipeline-CLI`
*   **Manual Trigger**: `aws codepipeline start-pipeline-execution --name MyNodePipeline-CLI`
*   **Update Pipeline**: Modify your JSON and run `aws codepipeline update-pipeline --cli-input-json file://pipeline.json`

---

## 6. Verify the CI/CD Workflow

---

## 4. Advanced: Adding a Manual Approval (Optional)

If you have a **Production** environment, you can add a manual approval step before deploying to prod:

1.  In the CodePipeline dashboard, click **Edit**.
2.  Add a new stage between **Dev** and **Prod**.
3.  Add an **Action** of type **Manual approval**.
4.  The pipeline will pause after the Dev deployment and wait for a human to click "Approve" before continuing to Production.
