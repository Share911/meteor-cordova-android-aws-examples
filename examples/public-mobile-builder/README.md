# Public Mobile Builder

Using AWS CodeBuild and a public ECR image.
This example will guide you through building an Meteor Android app in CodeBuild.
This example will use source code from a public git repository.

The example project used here is an empty meteor project running Meteor 2.4  
The example is configured to output a debug apk

Current builder is targeted for Cordova 10 and Meteor 2.2+

You can check the public base image used [here](https://github.com/Share911/meteor-cordova-android-public-image)
## Setup AWS Services
Using an AWS root account is not recommended (by AWS) for this project.  
But it is easier to start the project as a root user in order to setup proper restrictions for the custom user later on.  

### 1. Create a S3 Bucket for build output
Create a [S3 bucket](https://s3.console.aws.amazon.com/s3/home) that will hold your mobile build output  
Bucket versioning is recommended to preserve old builds, but it is not necessarily needed for this example.

### 2. Create a Codebuild Project
Create a CodeBuild Project for Mobile Builder with the following configurations:
1. Project Name: `public-mobile-builder`
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
         * GIT_CLONE_URL = https://github.com/Share911/empty-meteor-public.git or your meteor project repository url for git clone
         * GIT_BRANCH = main or Git branch to start your build on
         * SERVER = https://richie-empty-app.meteorapp.com or Your meteor webserver
         * APP_DIR = leave it empty for this example. This should be the path to your app, if it is not located in root of your repository
11. Build specifications: Use a buildspec file
12. Buildspec name: examples/public-mobile-builder/buildspec.yml
13. Artifact Type: Amazon S3
14. Bucket Name: `Your S3 Bucket Name`
15. Artifacts packaging: Zip
16. It is recommended to enable CloudWatch so you can see the logs for debugging.


### 3. Create Policies

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
2. Service: S3
    * ListBucket
    * ListAllMyBuckets
    * CreateBucket
    * GetObject
    * PutObject
3. Service: STS
    * GetServiceBearerToken

Once created, attach the policy into your codebuild role  
The role is usually named as codebuild-$projectname-service-role  

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
