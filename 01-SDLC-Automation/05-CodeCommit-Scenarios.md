# AWS CodeCommit: Real-World Scenarios and Troubleshooting

This guide covers common challenges faced by DevOps engineers when working with AWS CodeCommit and provides actionable solutions.

---

## 1. Authentication & Connection Problems

### Problem: 403 Forbidden Error (CLI/Git)
**Scenario**: You try to clone or push, and receive an `Access denied` or `403 Forbidden` error.
*   **Cause**: The IAM User/Role lacks the necessary CodeCommit permissions (e.g., `codecommit:GitPull`, `codecommit:GitPush`).
*   **Solution**:
    1.  Check the IAM Policy attached to the user. Ensure it has `AWSCodeCommitPowerUser` or a custom policy with the required actions.
    2.  If using **VPC Endpoints**, ensure the Security Group allows inbound traffic from your local machine/CI server.

### Problem: SSH "Permission Denied (publickey)"
**Scenario**: You've uploaded an SSH key to IAM but still cannot connect.
*   **Cause**: Your SSH client isn't sending the **SSH Key ID** as the username.
*   **Solution**:
    1.  Ensure your `~/.ssh/config` file is correctly configured:
        ```text
        Host git-codecommit.*.amazonaws.com
          User APK... (Your SSH Key ID from IAM)
          IdentityFile ~/.ssh/id_rsa (Your private key)
        ```
    2.  Verify the SSH Key ID matches exactly what is in the IAM console.

### Problem: HTTPS Prompt Loop
**Scenario**: Git keeps asking for your username and password repeatedly on Windows.
*   **Cause**: The **Git Credential Manager** is not storing the service-specific credentials correctly.
*   **Solution**:
    1.  Clear existing entries from **Windows Credential Manager**.
    2.  Ensure you are using the **HTTPS Git Credentials** generated in the IAM console, NOT your AWS console login or IAM access keys.

---

## 2. Operational Scenarios

### Problem: Large File Push Fails
**Scenario**: Pushing a large binary file (e.g., 200MB) results in a timeout or hang.
*   **Cause**: CodeCommit has a default file size limit for individual files and a total push size limit.
*   **Solution**:
    1.  Use **Git LFS (Large File Storage)** for large binaries. CodeCommit supports Git LFS natively.
    2.  Increase the Git buffer size: `git config http.postBuffer 524288000` (500MB).

### Problem: Merge Conflicts the UI Cannot Resolve
**Scenario**: The CodeCommit console says a PR cannot be merged due to conflicts, but doesn't offer a way to fix them.
*   **Cause**: Complex conflicts (e.g., binary changes or multiple line overlaps) often cannot be handled by simple web editors.
*   **Solution**:
    1.  Resolve locally:
        ```bash
        git checkout main
        git pull origin main
        git checkout feature-branch
        git merge main
        # Fix conflicts in your editor, then add/commit
        git push origin feature-branch
        ```
    2.  Refresh the PR; it should now show as mergable.

---

## 3. Permission & Access Scenarios

### Problem: Tag-Based Access Control Not Working
**Scenario**: You want to allow users to push only to repositories with a specific tag (e.g., `Project: Finance`), but they are getting denied.
*   **Cause**: Tag-based conditions in IAM policies often require the `aws:ResourceTag` prefix and the repository must be tagged BEFORE the policy is evaluated.
*   **Solution**:
    1.  Ensure the IAM policy uses: `"Condition": {"StringEquals": {"aws:ResourceTag/Project": "Finance"}}`.
    2.  Verify that the CodeCommit repository itself has the `Project` tag with the value `Finance`.

---

## 4. Integration Scenarios

### Problem: EventBridge Trigger Failing to Fire
**Scenario**: You've set up an EventBridge rule to trigger a Lambda on PR changes, but nothing happens.
*   **Cause**: The event pattern is too restrictive or the Lambda lacks resource-based permissions to be invoked by EventBridge.
*   **Solution**:
    1.  Simplify the event pattern to just `{"source": ["aws.codecommit"]}` to see if ANY events trigger.
    2.  Check the Lambda function's **Resource-based policy** to ensure `events.amazonaws.com` has `lambda:InvokeFunction` permission.

---

## 6. CodeCommit Interview Questions & Answers

### Q1: What is AWS CodeCommit and how does it differ from GitHub?
**Answer**: AWS CodeCommit is a managed source control service that hosts private Git repositories. Unlike GitHub (which is a standalone platform), CodeCommit is deeply integrated into the AWS ecosystem (IAM, KMS, CloudWatch). Key differences include:
*   **Authentication**: GitHub uses individual accounts; CodeCommit uses IAM.
*   **Storage**: CodeCommit scales automatically; no need to worry about repository sizes locally on a server.
*   **Encryption**: CodeCommit automatically encrypts files at rest using AWS KMS.

### Q2: How can you restrict a developer from deleting a branch in CodeCommit?
**Answer**: This is achieved using an **IAM Policy** with an explicit `Deny` effect on the `codecommit:DeleteBranch` action. You can refine this by using `Condition` blocks to target specific branches (like `refs/heads/main`).

### Q3: Explain the process of migrating an existing Git repository to CodeCommit.
**Answer**: The best practice is the **Mirror Method**:
1.  Create an empty repo in CodeCommit.
2.  Run `git clone --mirror <source-url>` to get all history and branches.
3.  Run `git push --mirror <codecommit-url>` to move everything to AWS.
4.  Optionally, use the `git-remote-codecommit` helper for easier authentication.

### Q4: What are Approval Rule Templates and why are they important?
**Answer**: Approval Rule Templates are automated configurations that apply review requirements to all new Pull Requests in a repository. They are critical for:
*   **Consistency**: Ensuring every PR follows the same quality standards.
*   **Security**: Preventing merges without at least one or two senior approvals.
*   **Automation**: Saving time by not having to manually add rules to every single PR.

### Q5: How do you implement Cross-Region Replication (CRR) for CodeCommit?
**Answer**: Since CodeCommit doesn't have a native "replication" button, it is implemented using a serverless architecture:
1.  **EventBridge**: Detects a repository state change (push/update) in the source region.
2.  **Lambda**: Triggered by EventBridge, it clones the repo from the source and pushes it to a repository in the destination region using a `--mirror` command.

### Q6: What is the difference between CodeCommit Triggers and Notifications?
**Answer**:
*   **Triggers**: Are used for **System-to-System** automation (e.g., triggering a Lambda or SNS to start a specialized workflow).
*   **Notifications**: Are used for **Human-to-Human** communication (e.g., sending an email or Slack message when a PR is created).

### Q7: If a developer gets a "403 Forbidden" error while pushing, how do you troubleshoot?
**Answer**:
1.  Verify the IAM role/user has `codecommit:GitPush` permissions.
2.  Check if there's a specific `Deny` policy on that branch.
3.  Ensure the developer is using the correct **HTTPS Git Credentials** or a properly configured **SSH Key**.
4.  Check if local AWS profile/credentials have expired.

---

## 7. Conclusion
This guide provides a bridge between theoretical knowledge and practical, real-world DevOps problems, helping you prepare for both daily operations and technical interviews.
