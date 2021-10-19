ARG BASE_IMAGE
ARG DESTINATION_DIR=mobile/releases

ARG PROJECT_PATH=/home/meteor/project
ARG DESTINATION_PATH=/home/meteor/$DESTINATION_DIR

# Using the base image built previously
FROM $BASE_IMAGE AS build-stage

# Git private key
ARG GITHUB_SSH
# Host URL of meteor app webserver
ARG SERVER
# Directory of the app within the repo
ARG APP_DIR

ARG PROJECT_PATH
ENV APP_PATH=$PROJECT_PATH/$APP_DIR
ARG DESTINATION_PATH

USER meteor

# Setup git ssh
RUN mkdir /home/meteor/.ssh && chmod 0700 /home/meteor/.ssh && ssh-keyscan -t rsa github.com >> /home/meteor/.ssh/known_hosts
RUN echo "$GITHUB_SSH" > /home/meteor/.ssh/id_rsa && chmod 400 /home/meteor/.ssh/id_rsa

# Prepare for build, should be done as meteor user
WORKDIR ${APP_PATH}
RUN git pull
RUN npm install

# Build Cordova Android app and point to your webserver
# This will build an .aab for publishing to Play Store
RUN meteor build $DESTINATION_PATH --platforms=android --server=$SERVER

# Use the line below to create an apk for debugging
# RUN meteor build $DESTINATION_PATH --platforms=android --server=$SERVER --packageType=apk --debug

# Outputs the build result into Codebuild container
FROM scratch AS export-stage

ARG DESTINATION_PATH
ENV OUTPUT_PATH $DESTINATION_PATH/android

COPY --from=build-stage $OUTPUT_PATH/README $OUTPUT_PATH/*.apk $OUTPUT_PATH/*.aab /

# Copying README because at least one file must exist
# https://stackoverflow.com/a/46801962/16994483