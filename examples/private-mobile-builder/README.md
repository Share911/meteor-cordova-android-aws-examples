# Private Mobile Builder

Using AWS CodeBuild and a public ECR image.
This example will guide you through building an Meteor Android app in CodeBuild.
This example will use source code from a private git repository.

The example project used here is an empty meteor project running Meteor 2.4  
The example is configured to output an unsigned aab

Current builder is targeted for Cordova 10 and Meteor 2.2+  

You can check the public base image used [here](https://github.com/Share911/meteor-cordova-android-public-image)

## Setup AWS Services
Using an AWS root account is not recommended (by AWS) for this project.  
But it is easier to start the project as a root user in order to setup proper restrictions for the custom user later on.  

### 1. Create a S3 Bucket for build output
Create a [S3 bucket](https://s3.console.aws.amazon.com/s3/home) that will hold your mobile build output  
Bucket versioning is recommended to preserve old builds, but it is not necessarily needed for this example.

### 2. Upload private key for git
This private key will be used to fetch your meteor project from a private git repository.
To do this:
1. Open AWS Secrets Manager
2. Store a new secret
3. Choose "Other type of secrets"
4. Use plaintext and paste your private key
5. Store your key
6. Note down the Secret ARN as it will be used in the next step

### 2. Create a Codebuild Project
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
         * GIT_CLONE_SSH_URL = your private meteor git repo SSH url (ex: git@github.com/acme/app.git)
         * GIT_BRANCH = main or Git branch to start your build on
         * SERVER = your meteor webserver url (ex: https://acme-app.meteorapp.com)
         * APP_DIR = leave it empty for this example. This should be the path to your app, if it is not located in root of your repository
         * GITHUB_SSH = Your git Secret ARN. Type = Secrets Manager
11. Build specifications: Use a buildspec file
12. Buildspec name: examples/private-mobile-builder/buildspec.yml
13. Artifact Type: Amazon S3
14. Bucket Name: `Your S3 Bucket Name`
15. Artifacts packaging: Zip
16. It is recommended to enable CloudWatch so you can see the logs for debugging.

### 4. Create Policies

#### CodeBuild Policy
These permissions are needed for CodeBuild to work with other AWS services.  
You can call this private-mobile-builder-codebuild policy.

Copy the JSON below to your policy JSON
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "ecr-public:DescribeImages",
                "ecr-public:DescribeRepositories",
                "ecr-public:GetAuthorizationToken",
                "s3:ListAllMyBuckets",
                "ecr-public:ListTagsForResource",
                "sts:GetServiceBearerToken",
                "ecr-public:GetRepositoryPolicy",
                "ecr-public:BatchCheckLayerAvailability"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue",
                "s3:ListBucket"
            ],
            "Resource": [
                "git-secret-arn", // Replace this with your Secret arn
                "s3-bucket-arn" // Replace this with your S3 bucket arn
            ]
        }
    ]
}
```
Once created, attach the policy into your codebuild role  
It is usually named as codebuild-$projectname-service-role  

## Start a mobile build
Once everything has been configured, head over to your codebuild project and click on start build.
Once the build is finished, you can find the exported result in your S3 directory.
