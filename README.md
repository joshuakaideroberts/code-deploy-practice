CodeDeploy auto-deploy from GitHub Actions (GitHub revision)

This repository contains a GitHub Actions workflow that triggers an AWS CodeDeploy deployment on pushes to the `main` branch. The deployment uses a GitHub revision: CodeDeploy pulls the code directly from this repository at the specific commit that triggered the workflow.

Required repository settings
- Secrets
  - AWS_ACCESS_KEY_ID — IAM user access key id
  - AWS_SECRET_ACCESS_KEY — IAM user secret
- Variables (Repository → Settings → Variables) or Environment variables
  - AWS_REGION — AWS region (for example: us-east-1)
  - CODEDEPLOY_APPLICATION — CodeDeploy application name
  - CODEDEPLOY_DEPLOYMENT_GROUP — CodeDeploy deployment group name

CodeDeploy ↔ GitHub connection
- In the AWS Console, connect your CodeDeploy application to GitHub so it can fetch the repository and commit. See AWS docs: Tutorials: Use CodeDeploy to deploy an application from GitHub.

Suggested minimal IAM permissions (no S3 required)

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:CreateDeployment",
        "codedeploy:GetApplication",
        "codedeploy:GetDeployment"
      ],
      "Resource": "*"
    }
  ]
}

How it works
- On push to `main`, Actions checks out the repo.
- The workflow builds a small JSON describing a GitHub revision (owner/repo + commit SHA) and calls `aws deploy create-deployment`.
- A follow-up step waits until the deployment completes (succeeds, fails, stopped, or timeout), and fails the job if the deployment fails or times out.

Notes
- Because CodeDeploy pulls this repo at the exact commit, there’s no separate artifact packaging or S3 bucket involved.
- If you need to inject build outputs or metadata into the deployed artifact, switch to an S3-based revision so the workflow can package a custom bundle.
