# syntax=docker/dockerfile:experimental

# Using cordova 10
FROM beevelop/cordova:v2021.04.1

# RUN apt-get update -qq
# Adding install ca-certificates here because of expired nodesource issue
# https://github.com/nodesource/distributions/issues/1266#issuecomment-931550203
RUN apt-get update ; apt-get install ca-certificates \
    && apt-get update \
    && apt-get install -y
RUN apt-get install -y apt-transport-https ca-certificates software-properties-common

# Add bzip2 and curl to unpack meteor installer
RUN apt-get install -y bzip2 curl tar

# Add git for pulling the app source from a git repo
RUN apt-get install -y git

# Add java8 repository
RUN apt-get install -y openjdk-8-jdk

# Add these to build native npm modules
RUN apt-get install -y --no-install-recommends python2.7 make g++ build-essential
ENV PYTHON=/usr/bin/python2.7

# Install latest Meteor as root
RUN curl -v https://install.meteor.com -o /tmp/install_meteor.sh
RUN sed -i.bak "s/tar -xzf.*/tar -xf \"\$TARBALL_FILE\" -C \"\$INSTALL_TMPDIR\"/g" /tmp/install_meteor.sh
RUN sh /tmp/install_meteor.sh

# Accept android studio licenses
RUN yes | ${ANDROID_SDK_ROOT}/tools/bin/sdkmanager --licenses

# Update and install Android SDKs using sdkmanager
RUN ${ANDROID_SDK_ROOT}/tools/bin/sdkmanager "tools" "platform-tools" --sdk_root=${ANDROID_SDK_ROOT} && \
    ${ANDROID_SDK_ROOT}/tools/bin/sdkmanager "build-tools;30.0.0" --sdk_root=${ANDROID_SDK_ROOT} && \
    ${ANDROID_SDK_ROOT}/tools/bin/sdkmanager "platforms;android-30" --sdk_root=${ANDROID_SDK_ROOT} && \
    ${ANDROID_SDK_ROOT}/tools/bin/sdkmanager "extras;android;m2repository" "extras;google;m2repository" --sdk_root=${ANDROID_SDK_ROOT}


# Builds android once
ARG DESTINATION_DIR=mobile/releases

ARG PROJECT_PATH=/home/meteor/project
ARG DESTINATION_PATH=/home/meteor/$DESTINATION_DIR
# Git branch and URL to clone via SSH
ARG GIT_CLONE_SSH_URL
ARG GIT_BRANCH
# Git private key
ARG GITHUB_DEPLOY_KEY
# Host URL of meteor app webserver
ARG SERVER
# Directory of the app within the repo
ARG APP_DIR

ENV APP_PATH=$PROJECT_PATH/$APP_DIR

# Create meteor user and give meteor user access to home folder
RUN useradd --groups users --create-home --shell /bin/bash meteor
WORKDIR /home/meteor

# Setup git ssh
RUN mkdir ~/.ssh && chmod 0700 ~/.ssh && ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

# Pull meteor app repository as root, as only root has access to the host ssh
RUN --mount=type=ssh git clone -b ${GIT_BRANCH} ${GIT_CLONE_SSH_URL} ${PROJECT_PATH}
RUN chown -R meteor /home/meteor

# As meteor user, call meteor --version to install meteor for this user
# This sets permissions properly for meteor user
USER meteor
RUN meteor --version

# Prepare for build, should be done as meteor user
WORKDIR ${APP_PATH}
RUN npm install
RUN meteor remove-platform ios ; exit 0
RUN meteor add-platform android ; exit 0
RUN mkdir -p ${DESTINATION_PATH}

# Build Cordova Android app and point to your webserver
# This will build an .aab for publishing to Play Store
RUN meteor build $DESTINATION_PATH --platforms=android --server=$SERVER

# Use the line below to create an apk for debugging
# RUN meteor build $DESTINATION_PATH --platforms=android --server=$SERVER --packageType=apk --debug
