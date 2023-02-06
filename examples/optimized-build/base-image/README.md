# Optimized Base Image Setup

Using AWS CodeBuild and ECR.
This example will guide you through building an optimized Meteor android app builder base image.

The example project used here is an empty meteor project running Meteor 2.4  

Current builder is targeted for Cordova 10 and Meteor 2.2+  

## Base Image
Using AWS CodeBuild, ECR, and Secrets Manager.
This will build a private base image that has your Meteor project built once.
This will reduce the time needed for subsequent builds.

## Setup AWS Services
Using an AWS root account is not recommended (by AWS) for this project.  
But it is easier to start the project as a root user in order to setup proper restrictions for the custom user later on.  

### 1. Create a private ECR Repository
This repository will contain a docker image which has Cordova installed .

Create a new [ECR repository](https://console.aws.amazon.com/ecr/home) with the following configurations:
1. Visibility Settings: Private
2. ECR Repository Name: `optimized-base-image`
3. Note down the repository name as it will be used in step 3

### 2. Grant access to private repo

#### 1. Create new public/private keypair

Source: [Instructions from Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

`$ ssh-keygen -t ed25519 -C "your_email@example.com"`

For this demo, we won't be adding a passphrase.  If you do want to use one then you can add another Secrets Manager resource for the passphrase and update the scripts to use it.

#### 2. Add public key as deploy key to Meteor app Github repo

Generate a deploy key for your Meteor app's private repo by going to Settings => Deploy Keys.
You should paste the _public_ key created in step 1 here.

#### 3. Add private key to Secrets Manager:
    1. Open AWS Secrets Manager
    2. Store a new secret
    3. Choose "Other type of secrets"
    4. Use plaintext and paste your _private_ key here
    5. Store your key
    6. Note down the Secret ARN as it will be used in the next step

### 3. Create a Codebuild Project
Create a CodeBuild Project for Mobile Builder with the following configurations:
1. Project Name: `optimized-base-image`
2. Source: The source here is how you will provide this git repository to AWS. For this example we will use GitHub.
3. Setup connection with your GitHub and choose this public repository.
4. Environment Image: Managed Image
5. Operating System: Ubuntu
6. Runtime(s): Standard
7. Image: aws/codebuild/standard:4.0
8. Privileged: Enabled
9. Service Role: New Service Role
10. Additional Configuration: 
     1. Compute: 7 GB, 4 vCPUs
     2. Setup the following Environment Variables
         * AWS_DEFAULT_REGION = your region-ID
         * IMAGE_TAG = latest
         * GIT_CLONE_SSH_URL = your private meteor git repo (ex: git@github.com/acme/app.git)
         * GIT_BRANCH = main or Git branch to start your build on
         * SERVER = your meteor webserver url (ex: https://acme-app.meteorapp.com)
         * APP_DIR = leave it empty for this example. This should be the path to your app, if it is not located in root of your repository
         * GITHUB_DEPLOY_KEY = ARN of your github deploy key stored in Secrets Manager. Type = Secrets Manager
         * AWS_ACCOUNT_ID = Your AWS account id
         * IMAGE_REPO_NAME = Your ECR repository name
11. Build specifications: Use a buildspec file
12. Buildspec name: examples/optimized-build/base-image/buildspec.yml
13. It is recommended to enable CloudWatch so you can see the logs for debugging.

### 4. Create Policies

#### CodeBuild Policy
These permissions are needed for CodeBuild to work with other AWS services.  
You can call this optimized-mobile-builder-codebuild policy.

Copy the JSON below to your policy JSON
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:DescribeImageScanFindings",
                "ecr:GetDownloadUrlForLayer",
                "ecr:ListTagsForResource",
                "s3:ListBucket",
                "ecr:UploadLayerPart",
                "ecr:ListImages",
                "ecr:PutImage",
                "secretsmanager:GetSecretValue",
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeImages",
                "ecr:DescribeRepositories",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetRepositoryPolicy"
            ],
            "Resource": [
                "secret-arn", // Replace with your Secret arn
                "ecr-arn", // Replace with your ECR arn, the format is arn:aws:ecr:$region:$account_id:repository/$repository_name
                "s3-bucket-arn" // Replace with your S3 bucket arn
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "ecr:GetAuthorizationToken"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "s3-obj-arn" // Replace with your S3 bucket arn with /* at the end $s3-bucket-arn/*
        }
    ]
}
```
Once created, attach the policy into your codebuild role  
It is usually named as codebuild-$projectname-service-role  

## Start the build
Once everything has been configured, head over to your codebuild project and click on start build.
Once the build is finished, CodeBuild will push your base image to ECR. You can then proceed with the mobile build.
