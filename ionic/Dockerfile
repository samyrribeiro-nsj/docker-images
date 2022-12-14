FROM openjdk:16-alpine

LABEL maintainer="Devops Nasajon <devops@nasajon.com.br>"

ENV GLIBC_VERSION 2.29-r0
ENV ANDROID_SDK_VERSION 9123335
ENV BUILD_TOOLS 32.0.0
ENV TARGET_SDK 32
ENV GRADLE_VERSION 7.6

ENV JAVA_HOME /opt/openjdk-16
ENV ANDROID_HOME /opt/android-sdk-linux
ENV ANDROID_SDK_ROOT /opt/android-sdk-linux
ENV GRADLE_HOME /opt/gradle-${GRADLE_VERSION}
ENV CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL file://${GRADLE_HOME}-bin.zip

ENV PATH ${PATH}:${JAVA_HOME}/bin:${ANDROID_SDK_ROOT}/tools/bin:${ANDROID_SDK_ROOT}/platform-tools:${ANDROID_SDK_ROOT}/emulator:${ANDROID_SDK_ROOT}/build-tools:${GRADLE_HOME}/bin

RUN apk add -U --no-cache \
    build-base \
    curl \
    git \
    npm \
    openssh \
    python3 \
    cairo-dev \
    giflib-dev \
    jpeg-dev \
    pango-dev \
 && npm -g config set user $USER \
 && npm i -g --unsafe-perm ionic cordova native-run cordova-set-version \
 && ionic --no-interactive config set -g daemon.updates false \
 && cordova telemetry off

# Download and install glibc
RUN echo "Downloading glibc" \
 && curl -SLo /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
 && curl -SLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-${GLIBC_VERSION}.apk \
 && curl -SLO https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-bin-${GLIBC_VERSION}.apk \
 && apk add --no-cache glibc-${GLIBC_VERSION}.apk glibc-bin-${GLIBC_VERSION}.apk \
 && rm -f /etc/apk/keys/sgerrand.rsa.pub glibc-${GLIBC_VERSION}.apk glibc-bin-${GLIBC_VERSION}.apk

# Download and extract Android Tools
RUN echo "Downloading Android Tools" \
 && curl -SLO https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip \
 && mkdir -p ${ANDROID_SDK_ROOT} \
 && unzip -d ${ANDROID_SDK_ROOT} commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip \
 && mkdir -p ${ANDROID_SDK_ROOT}/latest \
 && rm -f commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip

# Install SDK Packages
RUN echo "Installing Android Tools" \
 && mkdir -p ~/.android/ && touch ~/.android/repositories.cfg \
 && mv opt/android-sdk-linux/cmdline-tools/* opt/android-sdk-linux/latest \
 && mv opt/android-sdk-linux/latest opt/android-sdk-linux/cmdline-tools/latest \
 && yes | ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager "--licenses" \
 && ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager  "--update" \
 && ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager "platform-tools" "build-tools;${BUILD_TOOLS}" "platforms;android-${TARGET_SDK}" "extras;android;m2repository" "extras;google;instantapps" "extras;google;m2repository"

# Download and extract Gradle
RUN echo "Downloading Gradle" \
 && curl -SLO https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip \
 && mkdir -p ${GRADLE_HOME} \
 && unzip -d /opt gradle-${GRADLE_VERSION}-bin.zip \
 && mv gradle-${GRADLE_VERSION}-bin.zip ${GRADLE_HOME}-bin.zip \
 && gradle -v

RUN apk del curl \
 && mkdir -p $HOME/.android/avd \
 && mkdir /ionic

WORKDIR /ionic

# Default port
EXPOSE 8100

# Ionic Lab port
EXPOSE 8200

# Live Reload port
EXPOSE 35729

# Dev Logger port
EXPOSE 53703

CMD ["ionic", "serve", "-l", "--devapp", "--address", "0.0.0.0", "--lab-host", "0.0.0.0"]