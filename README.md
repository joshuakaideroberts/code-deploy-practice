CodeDeploy auto-deploy from GitHub Actions (S3 artifact)

This repository contains a GitHub Actions workflow that triggers an AWS CodeDeploy deployment on pushes to the `main` branch. The deployment uses an S3 artifact: the workflow packages the repo (including `appspec.yml`, hooks, and a small `deploy-info.json` file), uploads it to S3, and tells CodeDeploy to deploy that bundle.

Required repository settings
- Secrets
  - AWS_ACCESS_KEY_ID — IAM user access key id
  - AWS_SECRET_ACCESS_KEY — IAM user secret
  - S3_BUCKET — Name of the S3 bucket to upload artifacts to (for example: my-codedeploy-artifacts)
- Variables (Repository → Settings → Variables) or Environment variables
  - AWS_REGION — AWS region (for example: us-east-1)
  - CODEDEPLOY_APPLICATION — CodeDeploy application name
  - CODEDEPLOY_DEPLOYMENT_GROUP — CodeDeploy deployment group name
  - S3_PREFIX — Optional key prefix ("folder") for artifacts in the bucket (for example: codedeploy/artifacts)

Minimal IAM permissions for the GitHub Actions user

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:CreateDeployment",
        "codedeploy:GetApplication",
        "codedeploy:GetDeployment",
        "codedeploy:GetDeploymentGroup"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    }
  ]
}

How it works
- On push to `main`, Actions checks out the repo.
- The workflow generates `deploy-info.json` with the repository and commit hash, then zips the repo (excluding `.git` and `.github/`).
- The zip is uploaded to `s3://$S3_BUCKET/` and a CodeDeploy deployment is created pointing at that S3 object.
- A follow-up step polls until the deployment completes (succeeds, fails, stopped, or timeout). The job fails if the deployment fails or times out.

Runtime notes
- `appspec.yml` maps both `index.html` and `deploy-info.json` to `/var/www/html/` so the page can display the deployed commit.
- Hooks install and start Apache (`httpd`) and stop it on ApplicationStop via the scripts in `scripts/`.
 - You can organize artifacts under a path by setting `S3_PREFIX`. The workflow normalizes leading/trailing slashes, so `codedeploy/artifacts` and `/codedeploy/artifacts/` behave the same.

Bucket setup tips
- Create a dedicated S3 bucket for artifacts. Block public access is fine; CodeDeploy and this workflow only need API access.
- Replace `YOUR_BUCKET_NAME` in the sample IAM with your bucket name, and set the `S3_BUCKET` secret to that same name.
 - S3 doesn’t have real folders; paths are part of the object key. This workflow uses the key `"$S3_PREFIX/$ARTIFACT_NAME"` when `S3_PREFIX` is set.
