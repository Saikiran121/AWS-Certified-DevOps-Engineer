# AWS CodeCommit Hands-On: Creating a Repository

This guide provides step-by-step instructions for creating a repository in AWS CodeCommit using both the AWS Management Console (UI) and the AWS CLI.

---

## 1. Prerequisites

Before you begin, ensure you have:
*   An **AWS Account**.
*   **IAM Permissions**: Your user/role must have the `AWSCodeCommitFullAccess` policy or equivalent permissions.
*   **AWS CLI**: Installed and configured on your local machine if you plan to use the CLI method.

---

## 2. Method 1: AWS Management Console (UI)

This is the easiest way for beginners to get started.

### Steps:
1.  **Sign in** to the AWS Management Console.
2.  **Navigate to CodeCommit**:
    *   Search for "CodeCommit" in the services search bar.
    *   Select **Source** -> **Repositories** from the left-hand navigation pane.
3.  **Create Repository**:
    *   Click the orange **Create repository** button.
4.  **Configure Settings**:
    *   **Repository name**: Enter a unique name (e.g., `MyDemoRepo`).
    *   **Description**: (Optional) Add a brief description of the project.
    *   **Tags**: (Optional) Add key-value pairs for resource management.
5.  **Review and Complete**:
    *   Click **Create**.
6.  **Success**: You will be redirected to the repository's "Getting Started" page where you can find clone URLs.

---

## 3. Method 2: AWS CLI

The CLI is faster for experienced users and can be easily scripted.

### Command:
Use the `create-repository` command to create a new repo.

```bash
aws codecommit create-repository \
    --repository-name my-cli-repo \
    --repository-description "This repository was created using the AWS CLI"
```

### Parameters:
*   `--repository-name`: (Required) The name of the repository (max 100 characters).
*   `--repository-description`: (Optional) A description of the repository.

### Sample Output:
If successful, the CLI returns a JSON object containing the repository metadata:

```json
{
    "repositoryMetadata": {
        "accountId": "123456789012",
        "repositoryId": "f7d73111-a87a-42fc-b952-example",
        "repositoryName": "my-cli-repo",
        "repositoryDescription": "This repository was created using the AWS CLI",
        "lastModifiedDate": 1738321200.0,
        "creationDate": 1738321200.0,
        "cloneUrlHttp": "https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-cli-repo",
        "cloneUrlSsh": "ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-cli-repo",
        "Arn": "arn:aws:codecommit:us-east-1:123456789012:my-cli-repo"
    }
}
```

---

---

## 4. Method 1: Adding a File via UI (Direct Upload/Create)

AWS CodeCommit allows you to manage files directly from the console without cloning the repository.

### Steps:
1.  **Navigate** to your repository in the CodeCommit console.
2.  Choose the branch where you want to add the file (default is `main`).
3.  Click the **Add file** dropdown button.
4.  Choose **Create file** or **Upload file**:
    *   **To Create**:
        *   Enter the file name: `index.html`.
        *   Paste your HTML content into the editor.
    *   **To Upload**:
        *   Click **Choose file** and select your local `index.html`.
5.  **Commit the change**:
    *   **Author name**: Enter your name.
    *   **Email address**: Enter your email.
    *   **Commit message**: e.g., "Initial commit of index.html".
6.  Click **Commit changes**.

---

## 5. Method 2: Adding a File via CLI (The Git Way)

The standard way to add files is via the Git lifecycle.

### Steps:
1.  **Create the file** locally in your cloned directory:
    ```bash
    echo "<h1>Welcome to AWS CodeCommit</h1>" > index.html
    ```
2.  **Add the file** to the staging area:
    ```bash
    git add index.html
    ```
3.  **Commit the changes**:
    ```bash
    git commit -m "Add index.html to the repository"
    ```
4.  **Push the changes** to AWS CodeCommit:
    ```bash
    git push origin main
    ```

### Alternative: CLI-only (put-file)
You can also use the AWS CLI to upload a file directly without Git:
```bash
aws codecommit put-file \
    --repository-name my-cli-repo \
    --branch-name main \
    --file-path index.html \
    --file-content fileb://index.html \
    --commit-message "Upload index.html using AWS CLI" \
    --name "Your Name" \
    --email "your@email.com"
```

---

---

## 6. Understanding Pull Requests (PRs)

A Pull Request is a request to merge code changes from one branch (source) into another (destination). It is the primary way teams review code, discuss changes, and ensure quality before integrating new features.

### Key Benefits:
*   **Code Review**: Team members can comment on specific lines of code.
*   **Approval Rules**: Require a specific number of approvals before merging.
*   **Conflict Detection**: CodeCommit automatically checks if the branches can be merged cleanly.

---

## 7. Method 1: Creating a PR via UI (AWS Console)

### Steps:
1.  **Create a feature branch** (if you haven't already):
    *   In the console, go to **Branches** -> **Create branch**.
    *   Name it `feature-update` and base it on `main`.
2.  **Add/Modify a file** in the `feature-update` branch (e.g., update `index.html`).
3.  **Initiate PR**:
    *   Navigate to **Pull requests** in the left sidebar.
    *   Click the orange **Create pull request** button.
4.  **Configure PR**:
    *   **Source branch**: `feature-update`.
    *   **Destination branch**: `main`.
    *   Click **Compare**.
5.  **Review and Submit**:
    *   Check the "Changes" tab to see the diff.
    *   Enter a **Title** (e.g., "Updated landing page") and **Description**.
    *   Click **Create pull request**.
6.  **Merge**: Once reviewed, click **Merge** and choose a merge strategy (Fast-forward, Squash, or Three-way merge).

---

## 8. Method 2: Creating a PR via CLI

### Command:
Use the `create-pull-request` command.

```bash
aws codecommit create-pull-request \
    --title "CLI Feature Update" \
    --description "Merging code from feature-update to main" \
    --targets repositoryName=my-cli-repo,sourceReference=feature-update,destinationReference=main
```

### Parameters:
*   `--title`: Short title for the PR.
*   `--targets`: Includes `repositoryName`, `sourceReference` (your branch), and `destinationReference` (usually `main`).

### Sample Output:
```json
{
    "pullRequest": {
        "pullRequestId": "1",
        "title": "CLI Feature Update",
        "description": "Merging code from feature-update to main",
        "pullRequestStatus": "OPEN",
        "authorArn": "arn:aws:iam::123456789012:user/DevUser",
        "targets": [
            {
                "repositoryName": "my-cli-repo",
                "sourceReference": "refs/heads/feature-update",
                "destinationReference": "refs/heads/main",
                "mergeBase": "f7d73111-...",
                "sourceCommit": "5a4b3c...",
                "destinationCommit": "1e2d3f..."
            }
        ],
        "creationDate": 1738323000.0,
        "lastActivityDate": 1738323000.0
    }
}
```

### Merging via CLI:
```bash
aws codecommit merge-pull-request-by-fast-forward \
    --pull-request-id 1 \
    --repository-name my-cli-repo
```

---

---

## 9. Understanding Commits

A commit is a snapshot of your repository's files and directories at a specific point in time. In Git-based systems like CodeCommit, a commit represents a permanent record of changes.

### Key Components of a Commit:
*   **SHA-1 Hash**: A unique 40-character identifier (e.g., `5a4b3c...`) generated from the content. Changing even one character in a file results in a completely different hash.
*   **Author**: The person who originally wrote the code.
*   **Committer**: The person who applied the commit (usually the same as the author, but can differ in PR workflows).
*   **Timestamp**: When the change was recorded.
*   **Commit Message**: A description provided by the developer explaining "the why" of the change.

### Use Cases for Commits:
*   **Traceability**: Track who changed what and when for auditing and compliance.
*   **Rollback**: Quickly revert your application to a previous stable state if a bug is introduced.
*   **Experimentation**: Use commits to checkpoint your work, allowing you to try new things and delete them if they don't work.
*   **Bug Discovery**: Tools like `git bisect` use commit history to identify exactly which commit introduced a bug.

### Key Features in AWS CodeCommit:
*   **Visual Diffs**: The AWS Console provides a side-by-side comparison of changes in any commit.
*   **Commit Graph**: Visualize the history of your repository, including branches and merges.
*   **GPG Signing Support**: You can sign your commits with a GPG key to verify the identity of the committer, ensuring the code hasn't been tampered with.
*   **Git Tags & Notes**: Associate human-readable labels (like `v1.0.0`) or additional metadata with specific commits.
*   **Integration with CloudWatch**: Use Amazon CloudWatch Events to monitor commit activity and trigger notifications or pipelines.

---

---

## 10. Understanding Git Tags

Git Tags are used to mark specific points in a repository's history as being important. Typically, they are used to capture release points (e.g., `v1.0.0`, `v2.1.0`).

### Types of Tags:
*   **Lightweight Tags**: Just a pointer to a specific commit. Think of it as a branch that doesn't change.
*   **Annotated Tags**: Stored as full objects in the Git database. They contain the tagger's name, email, date, and a tagging message. Recommended for public releases.

---

## 11. Managing Tags

### Method 1: AWS Management Console (UI)
1.  Navigate to your repository in the **CodeCommit console**.
2.  In the left sidebar, choose **Git tags**.
3.  Click the orange **Create tag** button.
4.  **Tag name**: Enter the version (e.g., `v1.0.0`).
5.  **Tag from**: Choose the branch (e.g., `main`) or enter a specific commit ID.
6.  **Description**: (Optional) Add details about the release.
7.  Click **Create tag**.

### Method 2: Git CLI
1.  **Create an annotated tag**:
    ```bash
    git tag -a v1.0.0 -m "Initial stable release"
    ```
2.  **Push the tag** to CodeCommit:
    ```bash
    # Push a specific tag
    git push origin v1.0.0

    # Push all local tags
    git push origin --tags
    ```
3.  **List tags**:
    ```bash
    git tag
    ```

---

---

## 12. Repository Settings & Administration

Managing your repository's configuration is essential for security and workflow automation.

### 12.1 General Settings
*   **Repository Name**: You can rename your repository here. Note that this will change the clone URLs.
*   **Description**: Update the metadata about your project.
*   **Delete Repository**: A destructive action. You must type the repository name to confirm deletion. **Use with extreme caution.**

### 12.2 Notifications
Notifications keep your team informed about repository activity using the **AWS User Notifications** service.
*   **Create Notification Rule**: Use this to send alerts for events like:
    *   Pull request status changes.
    *   Comments on commits or PRs.
    *   Branch or tag creation/deletion.
*   **Targets**: You can send these notifications to **SNS Topics** (Email, SMS) or **AWS Chatbot** (Slack, Chime).

### 12.3 Triggers
Triggers allow you to integrate CodeCommit with other AWS services automatically.
*   **SNS Triggers**: Send a message to an SNS topic whenever code is pushed.
*   **Lambda Triggers**: Execute a Lambda function on push events. This is useful for:
    *   Static analysis (linting).
    *   Integration with external systems (Webhooks).
    *   Triggering external build systems.
*   **Events**: You can trigger based on all branch activity or specific branch patterns.

### 12.4 Repository Settings
*   **Default Branch**: Change the branch that users see by default when they open the repository (usually `main`).
*   **Merge Strategy**: Configure the default merge strategy for Pull Requests (Fast-forward, Squash, or Three-way).
*   **Approval Rule Templates**: (Advanced) Create rules that apply to all repositories or specific ones, requiring a certain number of approvals for PRs.

---

---

## 13. Hands-On: Creating Notifications

Notifications allow you to receive alerts when specific events occur in your repository.

### Method 1: AWS Management Console (UI)
1.  Navigate to your repository in the **CodeCommit console**.
2.  In the left sidebar, choose **Settings** -> **Notifications**.
3.  Click the orange **Create notification rule** button.
4.  **Notification name**: Enter a name (e.g., `MyRepoNotify`).
5.  **Detail type**: Choose **Basic** (standard alerts) or **Full** (includes more metadata).
6.  **Events that trigger notifications**: Select events like:
    *   Pull request created.
    *   Commit merged.
    *   Comment on commit.
7.  **Targets**: 
    *   Choose **Create SNS topic** or select an existing one.
    *   Enter a topic name and confirm.
8.  Click **Submit**.

### Method 2: AWS CLI
Use the `codestar-notifications` service to manage notification rules.

**Command:**
```bash
aws codestar-notifications create-notification-rule \
    --name MyCLINotify \
    --event-type-ids codecommit-repository-comments-on-commits codecommit-repository-pull-request-created \
    --resource arn:aws:codecommit:us-east-1:123456789012:my-cli-repo \
    --targets TargetType=SNS,TargetAddress=arn:aws:sns:us-east-1:123456789012:MySNSTopic \
    --detail-type BASIC
```

**Parameters:**
*   `--event-type-ids`: A list of event IDs you want to monitor.
*   `--resource`: The Amazon Resource Name (ARN) of your repository.
*   `--targets`: The ARN of your SNS topic.

---

---

## 14. Hands-On: Creating Triggers

Triggers allow you to integrate CodeCommit with other services like SNS (for notifications) or Lambda (for automation) on specific repository events.

### Method 1: AWS Management Console (UI)
1.  Navigate to your repository in the **CodeCommit console**.
2.  In the left sidebar, choose **Settings** -> **Triggers**.
3.  Click the orange **Create trigger** button.
4.  **Trigger name**: Enter a name (e.g., `MyLambdaTrigger`).
5.  **Events**: Select events (e.g., **Push to existing branch**).
6.  **Branches**: Choose **All branches** or specify names.
7.  **Service details**:
    *   **SNS topic**: Select an existing SNS topic.
    *   **Lambda function**: Choose a Lambda function from your account.
8.  Click **Create trigger**.
9.  (Optional) Click **Test trigger** to verify the configuration.

### Method 2: AWS CLI
You can create triggers by providing a JSON configuration to the `put-repository-triggers` command.

**Command:**
```bash
aws codecommit put-repository-triggers \
    --repository-name MyRepo \
    --triggers name=MyCLITrigger,destinationArn=arn:aws:lambda:us-east-1:123456789012:function:MyTestFunction,events=all
```

**Parameters:**
*   `--repository-name`: The name of the repository.
*   `--triggers`: A list of trigger configurations (Name, Destination ARN, and Events).

**Verification:**
To view the triggers you just created:
```bash
aws codecommit get-repository-triggers --repository-name MyRepo
```

---

---

## 15. Comparison: Notifications vs. Triggers

It’s common to confuse Notifications and Triggers since both can use SNS. Here is the key breakdown to help you choose the right one:

| Feature | Notifications (AWS User Notifications) | Triggers (CodeCommit Native) |
| :--- | :--- | :--- |
| **Primary Audience** | **Humans** (Developers, DevOps Engineers) | **Systems** (Automation, CI/CD, Scripts) |
| **Core Goal** | "Tell me when something happened." | "Do something when something happened." |
| **Typical Targets** | Email, SMS, Slack, Microsoft Teams | AWS Lambda, SNS (for logic/fan-out) |
| **Best For** | Reviewing PRs, monitoring comments, alerts. | Triggering builds, auto-formatting, webhooks. |
| **Setup Location** | Settings -> Notifications | Settings -> Triggers |

### Which one should I use?
*   **Use Notifications** if you want to get an **email or a Slack message** when someone creates a Pull Request or leaves a comment on your code.
*   **Use Triggers** if you want to **automatically run a Lambda function** to scan your code for passwords (secrets scanning) or trigger a build in a non-AWS CI tool every time you push code.

---

---

## 16. Authentication for AWS CodeCommit

To interact with CodeCommit from your local machine, you must authenticate. There are two primary long-term methods: **SSH Public Keys** and **HTTPS Git Credentials**.

### 16.1 SSH Public Keys
**What it is:** Key-pair authentication where you provide your public key to AWS and keep the private key locally.
**Why we need it:**
*   **Security:** It’s more secure than passwords.
*   **Non-interactive:** Ideal for Linux/Mac/WSL users who want to avoid typing passwords repeatedly. It works seamlessly with `ssh-agent`.

**How to set up:**
1.  **Generate a key pair** locally: `ssh-keygen`.
2.  **Upload to IAM**:
    *   Go to the **IAM Console**.
    *   Select your **User** -> **Security credentials** tab.
    *   Scroll to **SSH public keys for AWS CodeCommit** and click **Upload SSH public key**.
    *   Paste your `.pub` file content.
3.  **Note the SSH Key ID**: Copy the ID starting with `APK...`.
4.  **Configure local SSH config**: Update your `~/.ssh/config` file to associate the ID with the AWS host.

### 16.2 HTTPS Git Credentials
**What it is:** A service-specific username and password generated by IAM.
**Why we need it:**
*   **Ease of Use:** Familar "Username/Password" flow.
*   **Cross-platform:** Works perfectly on Windows and with Git Credential Manager.
*   **Compatibility:** No need to configure SSH files.

**How to set up (UI Way):**
1.  Go to the **IAM Console**.
2.  Select your **User** -> **Security credentials** tab.
3.  Scroll to **HTTPS Git credentials for AWS CodeCommit**.
4.  Click **Generate credentials**.
5.  **Download/Copy**: Keep the credentials safe. They are only shown once.

### 16.3 SSH vs. HTTPS Comparison

| Feature | SSH | HTTPS |
| :--- | :--- | :--- |
| **Setup Complexity** | Moderate (Requires config file) | Low (Direct username/password) |
| **Security** | High (Key-based) | Moderate (Password-based) |
| **Best For** | Linux/Mac, CI Servers | Windows users, Beginners |

---

---

## 17. Hands-On: HTTPS Workflow (Clone, Add, Push)

Now that you have your **HTTPS Git credentials**, follow these steps to manage your code.

### Step 1: Copy the HTTPS Clone URL
1.  Navigate to your repository in the **CodeCommit console**.
2.  Click the **Clone URL** button in the top right.
3.  Select **Clone HTTPS**.

### Step 2: Clone the Repository
Open your terminal and run:
```bash
git clone <https-clone-url>
```

### Step 3: Authenticate
When prompted by Git, enter the **Username** and **Password** you generated in the IAM console (Step 16.2).
> [!TIP]
> On Windows, the **Git Credential Manager** will usually pop up a window to save these credentials so you don't have to enter them again.

### Step 4: Add a File
1.  Move into the repository directory:
    ```bash
    cd <repository-name>
    ```
2.  Create a new file (e.g., `README.md`):
    ```bash
    echo "# My Project Documentation" > README.md
    ```

### Step 5: Commit and Push
1.  **Stage** the file:
    ```bash
    git add README.md
    ```
2.  **Commit** the change:
    ```bash
    git commit -m "Add initial README"
    ```
3.  **Push** to CodeCommit:
    ```bash
    git push origin main
    ```

### Step 6: Verify in AWS Console
Refresh your repository page in the CodeCommit console. You should now see `README.md` in the file list.

---

---

## 18. Deep Dive: Cloning Methods

There are three ways to clone a CodeCommit repository. Choose the one that fits your workflow.

### 18.1 Method 1: HTTPS (Standard)
Best for beginners and Windows users.
```bash
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo
```
*   **Requires:** HTTPS Git credentials generated in IAM (Step 16.2).
*   **Prompt:** You will be asked for a username and password on the first clone.

### 18.2 Method 2: SSH
Best for Linux/Mac users and automated scripts.
```bash
git clone ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo
```
*   **Requires:** SSH Public Key uploaded to IAM (Step 16.1) and a local `~/.ssh/config` entry.

### 18.3 Method 3: HTTPS (GRC) - Recommended
**Git Remote CodeCommit (GRC)** is a utility that allows you to clone using your local AWS CLI profiles or IAM roles without needing separate Git credentials.

**Setup:**
```bash
pip install git-remote-codecommit
```

**Clone Command:**
```bash
# Uses default AWS profile
git clone codecommit://MyRepo

# Uses a specific AWS profile
git clone codecommit://MyProfile@MyRepo
```
*   **Advantage:** No passwords or SSH keys needed; it uses your temporary IAM session tokens.

---

## 19. Post-Creation: Verifying the Repository

Regardless of the method used, you can verify your repository exists by listing them:

**AWS CLI:**
```bash
aws codecommit list-repositories
```

**AWS Console:**
Simply refresh the **Repositories** page in the CodeCommit dashboard.

---

## 20. Conclusion

You have now mastered the basics of AWS CodeCommit, including repository creation, file management, pull requests, triggers, notifications, and secure authentication.
