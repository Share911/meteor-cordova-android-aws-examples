## About The Project

Building a Meteor Android app can take a lot of development time.
From setting up your local environment, downloading Android Studio, and the build time itself.

Using AWS, we were able to move our entire build process to their services.
No more setting up local environment and waiting for build to finish.

Using AWS CodeBuild, ECR, S3, and Secrets Manager.
This example will build an Android app from a Meteor project on AWS services.

The example project used here is an empty meteor project running Meteor 2.4
Current builder is targeted for Cordova 10 and Meteor 2.2+

## How it works

The main component of this build process is [AWS CodeBuild](https://aws.amazon.com/codebuild/).
CodeBuild will help us run our build script within their resources.
We just have to set it up once and it will be able to generate your latest Android build in a single click.

The output of CodeBuild (apk or aab) will be stored in S3.

To help reduce build time on CodeBuild, we will start our build on a pre-configured environment.
This is where [AWS ECR](https://aws.amazon.com/ecr/) comes in.
Using ECR, we can store a pre-configured docker image that will be used for our mobile build.

In addition to the 2 services above, we will utilize [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) to keep our sensitive informations.
For example, in order to pull from a private git repository, we will use a private key that's stored within Secrets Manager.

With these three services, we can build a Meteor Android app easily and even setup a continous integration.

## Examples

As every project has different need, we have provided 3 examples that might suit your needs.
1. [Build from a public git repository using our base image*](https://github.com/Share911/meteor-cordova-android-aws-examples/tree/main/examples/public-mobile-builder) (apk)
2. [Build from a private git repository using our base image*](https://github.com/Share911/meteor-cordova-android-aws-examples/tree/main/examples/private-mobile-builder) (aab)
3. [Build from a private git repository using a customized base image](https://github.com/Share911/meteor-cordova-android-aws-examples/tree/main/examples/optimized-build) (aab)

We recommend you to read #1 or #2 before starting with #3.
#3 is a very optimized build that will reduce build time significantly. It is recommended for continous integration.

*We have provided a public ECR docker image with a pre-configured environment that should be suitable to build your Meteor Android app.
You can check the public base image used [here](https://github.com/Share911/meteor-cordova-android-public-image)

## AWS User
We recommend you to use a root account to set up this project.
If you'd like to use an IAM user, make sure that it has ECR, ECR Public, CodeBuild, CloudWatch, Secrets Manager, and S3 permissions.
