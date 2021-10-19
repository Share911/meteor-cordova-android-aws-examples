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

### 2. Upload private key for git
This private key will be used to fetch your meteor project from a private git repository.
To do this:
1. Open AWS Secrets Manager
2. Store a new secret
3. Choose "Other type of secrets"
4. Use plaintext and paste your private key
5. Store your key
6. Note down the Secret ARN as it will be used in step 3

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
         * GIT_CLONE_URL = <your private meteor git repo> (ex: https://github.com/acme/app.git)
         * GIT_BRANCH = main or Git branch to start your build on
         * SERVER = <your meteor webserver url> (ex: https://acme-app.meteorapp.com)
         * APP_DIR = leave it empty for this example. This should be the path to your app, if it is not located in root of your repository
         * GITHUB_SSH = Your git Secret ARN. Type = Secrets Manager
         * AWS_ACCOUNT_ID = Your AWS account id
         * IMAGE_REPO_NAME = Your ECR repository name
11. Build specifications: Use a buildspec file
12. Buildspec name: examples/optimized-build/base-image/buildspec.yml
13. It is recommended to enable CloudWatch so you can see the logs for debugging.

### 4. Create Policies

#### CodeBuild Policy
These permissions are needed for CodeBuild to work with other AWS services.  
You can call this meteor-mobile-codebuild policy.  
The permissions are as follow:
1. Service: Elastic Container Registry
    * GetAuthorizationToken
    * BatchCheckLayerAvailability
    * GetDownloadUrlForLayer
    * GetRepositoryPolicy
    * DescribeRepositories
    * ListImages
    * DescribeImages
    * BatchGetImage
    * ListTagsForResource
    * DescribeImageScanFindings
    * InitiateLayerUpload
    * UploadLayerPart
    * CompleteLayerUpload
    * PutImage
    * Set specific resource to your ECR Repository
2. Service: Secrets Manager
    * GetSecretValue
    * Set specific resource to your Github private key
3. Service: S3
    * ListBucket
    * ListAllMyBuckets
    * CreateBucket
    * GetObject
    * PutObject

Once created, attach the policy into your codebuild role  
It is usually named as codebuild-$projectname-service-role  

## AWS User
Using an AWS root account is not recommended for this project.  
If you insist on using a root account, you can skip this step.  

#### User Policy
There are several permissions that are needed for an user to run this project.  
This single policy should allow user to handle every part of this project.  
You can call this meteor-mobile-builder policy.  
The permissions are as follow:
1. Code Build
    * ListProjects
    * ListBuilds
    * ListBuildsForProject
    * BatchGetProjects
    * BatchGetBuilds
    * RetryBuild
    * StartBuild
    * StopBuild
    * UpdateProject
    * Set specific resource to your base image and mobile builder projects
2. CloudWatch Logs
    * GetLogEvents
    * DescribeLogGroups
    * Set to all resources
2. S3
    * ListBucket
    * GetObject
    * GetObjectVersion
    * Set Specific resource to your output S3

### Setup AWS User
Create or use an existing AWS user and give them meteor-mobile-builder policy.

## Start the build
Once everything has been configured, head over to your codebuild project and click on start build.
Once the build is finished, you can find the exported result in your S3 directory.
