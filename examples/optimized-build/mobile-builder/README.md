# Optimized Mobile Builder Setup

Using AWS CodeBuild and your custom ECR image.
This example will guide you through building an Meteor Android app in CodeBuild.
This example will use source code from a private git repository.

The example project used here is an empty meteor project running Meteor 2.4  
The example is configured to output an unsigned aab

Current builder is targeted for Cordova 10 and Meteor 2.2+  

If you haven't setup your own custom ECR image, please start with that first.
[Build your optimized base image](https://github.com/Share911/meteor-cordova-android-aws-examples/tree/main/examples/optimized-build/base-image)

## Setup AWS Services

Using an AWS root account is not recommended (by AWS) for this project.  
But it is easier to start the project as a root user in order to setup proper restrictions for the custom user later on.  

As part of setting up the "base-image", you should already have an S3 bucket for output, secrets in Secrets Manager, and an IAM Policy.

### 1. Create a Codebuild Project
Create a CodeBuild Project for Mobile Builder with the following configurations:
1. Project Name: `private-mobile-builder`
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
         * STAGE = "prod" or "qa"
         * SERVER = your meteor webserver url (ex: https://acme-app.meteorapp.com)
         * APP_DIR = leave it empty for this example. This should be the path to your app, if it is not located in root of your repository
         * GITHUB_DEPLOY_KEY = ARN of your github deploy key stored in Secrets Manager. Type = Secrets Manager
         * GOOGLE_SERVICES_JSON = ARN of the google services json stored in Secrets Manager. Type = Secrets Manager
         * AWS_ACCOUNT_ID = Your AWS account id
         * IMAGE_REPO_NAME = ECR repository name of your optimized base image
11. Build specifications: Use a buildspec file
12. Buildspec name: examples/optimized-build/mobile-builder/buildspec.yml
13. Artifact Type: Amazon S3
14. Bucket Name: `Your S3 Bucket Name`
15. Artifacts packaging: Zip
16. It is recommended to enable CloudWatch so you can see the logs for debugging.

### 2. Attach Policy to Role

Attach the policy you created as part of the "base-image" steps to your new CodeBuild role.
  
It is usually named something like: codebuild-$projectname-service-role  

## Start a mobile build
Once everything has been configured, head over to your codebuild project and click on start build.
Once the build is finished, you can find the exported result in your S3 directory.
