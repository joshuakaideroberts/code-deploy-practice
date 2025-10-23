CodeDeploy auto-deploy from GitHub Actions

This repository contains a simple GitHub Actions workflow that packages the repository, uploads it to S3, and triggers an AWS CodeDeploy deployment when pushing to the `main` branch.

Required GitHub repository secrets
- AWS_ACCESS_KEY_ID — IAM user access key id
- AWS_SECRET_ACCESS_KEY — IAM user secret
- AWS_REGION — AWS region (for example: us-east-1)
- S3_BUCKET — S3 bucket name where artifacts will be uploaded
- CODEDEPLOY_APPLICATION — CodeDeploy application name
- CODEDEPLOY_DEPLOYMENT_GROUP — CodeDeploy deployment group name

Suggested minimal IAM permissions

The IAM user for the GitHub Action needs permission to upload to S3 and to create deployments in CodeDeploy. Example policy (adjust resources to your environment and least privilege):

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"s3:PutObject",
				"s3:PutObjectAcl",
				"s3:GetObject",
				"s3:ListBucket"
			],
			"Resource": [
				"arn:aws:s3:::YOUR_BUCKET_NAME",
				"arn:aws:s3:::YOUR_BUCKET_NAME/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"codedeploy:CreateDeployment",
				"codedeploy:GetApplication",
				"codedeploy:GetDeployment",
				"codedeploy:RegisterApplicationRevision"
			],
			"Resource": "*"
		}
	]
}

How it works
- On push to `main`, Actions checks out the repo.
- The workflow zips the repository (excluding `.github` and `.git`) and uploads the zip to the configured S3 bucket.
- It then calls CodeDeploy to create a deployment pointing to the uploaded S3 zip.

Testing locally
- You can test by creating a zip and uploading using the AWS CLI then calling `aws deploy create-deployment` as in the workflow.

Notes and next steps
- The workflow uses an S3-backed revision. Ensure your CodeDeploy application and deployment group are configured to use the S3 bundle type.
- For blue/green deployments or more complex pipelines, consider adding build/test steps and more advanced artifact versioning.
