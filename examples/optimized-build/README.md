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
