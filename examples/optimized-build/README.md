# Optimized Private Build Setup

Using AWS CodeBuild and ECR.
This example will guide you through building an optimized Meteor android app builder.

The example project used here is an empty meteor project running Meteor 2.4  
The example is configured to output an unsigned aab

Current builder is targeted for Cordova 10 and Meteor 2.2+  

This example consists of 2 parts:
1. Building a base image for your builder.
2. Setup a builder for mobile builds.

## Why we call it optimized?

This example use the advantage of building your base image yourself.
Inside your base image, we will build the mobile app once.
Doing this will reduce the build time of subsequent builds significantly.
## Base Image
Using AWS CodeBuild, ECR, and Secrets Manager.
This will build a private base image that already has your Meteor project built once.
This will reduce the time needed for subsequent builds.

## Mobile Builder
Using AWS CodeBuild, ECR, and Secrets Manager.
This will use your private base image and output an Android apk/aab.


The example project used here is an empty meteor project running Meteor 2.4 

Current builder is targeted for Cordova 10 and Meteor 2.2+  

## Setup AWS Services
Using an AWS root account is not recommended for this project.  
But it is easier to start the project as a root user in order to setup proper restrictions for the custom user later on.  

There are a few variables that are going to be reused between services. It will be good to note these down as go:
   * Region: Your AWS region.
   * ECR Repository Name: The ECR Repository name for your base image.
   * CodeBuild Base Image Name: CodeBuild project name for base image.
   * CodeBuild Mobile Builder Name: CodeBuild project name for mobile builder.
   * Git Private Key ARN: Secret ARN of your git private key
   * S3 Bucket Name

### 1. Create a S3 Bucket for build output
Create a [S3 bucket](https://s3.console.aws.amazon.com/s3/home) that will hold your mobile build output  
(It is recommended that you enable bucket versioning to preserve old builds.)

### 2. Create a Codebuild Project
Create a CodeBuild Project for Mobile Builder with the following configurations:
1. Project Name: `CodeBuild Mobile Builder Name`
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
11. Build specifications: Use a buildspec file
12. Buildspec name: mobile-builder/buildspec.yml
13. Artifact Type: Amazon S3
14. Bucket Name: `Your S3 Bucket Name`
15. Artifacts packaging: Zip
16. It is recommended to enable CloudWatch so you can see the logs for debugging.


### 3. Upload private key for git
This private key will be used to fetch your meteor project from a private git repository.
To do this:
1. Open AWS Secrets Manager
2. Store a new secret
3. Choose "Other type of secrets"
4. Use plaintext and paste your private key
5. Store your key
6. Note down the Secret ARN into `Git Private Key ARN`

### 4. Create Policies

#### CodeBuild Policy
These permissions are needed for CodeBuild to work with other AWS services.  
You can call this meteor-mobile-codebuild policy.  
The permissions are as follow:
1. Service: Elastic Container Public Registry
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
4. Service: STS
    * GetServiceBearerToken

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

## Start a mobile build
Once everything has been configured, head over to your codebuild project and click on start build.
Once the build is finished, you can find the exported result in your S3 directory.

## Optimization

### Store project code in private Docker image

The first time that you build your android app, Meteor will create a cache that will speed up subsequent builds.  We can take advantage of this by creating a new base docker image that has your specific project already built once in it.

Here are the steps to do this:
  1. Create a private ECR Repository
  2. Create a git repo (docker file + buildspec)
  3. Create a new CodeBuild project to create the new private build image
  4. Create another CodeBuild project to build your app

#### 1. Create a private ECR Repository
This repository will contain a docker image which has Cordova installed .

Create a new [ECR repository](https://console.aws.amazon.com/ecr/home) with the following configurations:
1. Visibility Settings: Private
2. ECR Repository Name: `<your_project>-android`

#### 2. Create a git repo

TODO

#### 3. Create a CodeBuild Project for the new private build image

1. Project Name: `CodeBuild Base Image Name`
2. Source: The source here is how you will provide this git repository to AWS. For simplicity we will use GitHub.
3. Setup connection with your GitHub and choose this repository.
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
         * AWS_ACCOUNT_ID = your aws account-ID
         * IMAGE_TAG = latest
         * IMAGE_REPO_NAME = `ECR Repository Name`
11. Build specifications: Use a buildspec file
12. Buildspec name: base-image/buildspec.yml
13. It is recommended to enable CloudWatch so you can see the logs for debugging.

#### 4. Create a CodeBuild project

TODO

## Base Image
Base Image is a simple image that uses Cordova as its base and installs android studio sdk and meteor on top of it.  

### Setting Up Base Image
Derived from https://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html  
Start build Base Image on CodeBuild  

## Mobile Builder
Using the generated Base Image, mobile builder will fetch your meteor repository and outputs a mobile build to an S3 bucket.  

### Setting Up Mobile Builder
Copy your `Git Private Key ARN` into GITHUB_SSH on mobile-builder/buildspec.yml  
Start build Mobile Builder on Codebuild  
Once the build has finished, check the outputs at your S3 bucket  
