# Use Ubuntu 20.04 as base image
FROM ubuntu:20.04

# Set timezone and locale
ENV TZ=Asia/Tehran
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

# Update package lists and install required dependencies
RUN apt-get update && apt install -y \
    tzdata \
    build-essential \
    g++ \
    gcc \
    python3-pip \
    unzip \
    curl \
    xz-utils \
    ca-certificates \
    software-properties-common \
    sudo \
    make \
    perl \
    libicu-dev \
    openjdk-17-jdk-headless \
    libxkbcommon-x11-0 \
    libfontconfig1 \
    libegl1 \
    adb \
    && ln -fs /usr/share/zoneinfo/$TZ /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && apt-get clean

# Install latest CMake
ARG CMAKE_VERSION=3.27.7
RUN curl -Lo install-cmake.sh https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.sh \
    && chmod +x install-cmake.sh \
    && ./install-cmake.sh --skip-license --prefix=/usr/local \
    && rm -fv install-cmake.sh

# Install aqtinstall for Qt installation
RUN pip install --upgrade aqtinstall

# Set Qt and Android versions
ARG QT_VERSION=6.5.2
ARG ANDROID_SDK_VERSION=9123335
ARG ANDROID_NDK_VERSION=25.1.8937393

# Install Qt 6.5.2 for both desktop and Android
RUN aqt install-qt linux desktop ${QT_VERSION} gcc_64 --outputdir /opt/qt --modules qtwebview qtquick3d qtquicktimeline qtremoteobjects qtserialport qtshadertools
RUN aqt install-qt linux android ${QT_VERSION} android_arm64_v8a --outputdir /opt/qt --modules qtwebview qtquick3d qtquicktimeline qtremoteobjects qtserialport qtshadertools

# Set environment variables
ENV ANDROID_SDK_ROOT=/opt/android-sdk
ENV ANDROID_NDK_ROOT=${ANDROID_SDK_ROOT}/ndk/${ANDROID_NDK_VERSION}
ENV QT_ANDROID_PATH=/opt/qt/${QT_VERSION}/android_arm64_v8a
ENV QT_HOST_PATH=/opt/qt/${QT_VERSION}/gcc_64
ENV PATH=${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/platform-tools:${ANDROID_SDK_ROOT}/build-tools/34.0.0:${QT_HOST_PATH}/bin:${QT_ANDROID_PATH}/bin:$PATH
ENV QT_SELECT=6

# Install Android SDK, NDK, Build Tools, and Gradle
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools \
    && curl -o sdk.zip https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip \
    && unzip sdk.zip -d ${ANDROID_SDK_ROOT}/cmdline-tools \
    && mv ${ANDROID_SDK_ROOT}/cmdline-tools/cmdline-tools ${ANDROID_SDK_ROOT}/cmdline-tools/latest \
    && rm sdk.zip

# Accept Android SDK licenses
RUN yes | ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager --sdk_root=${ANDROID_SDK_ROOT} --licenses

# Install required Android SDK components
RUN ${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin/sdkmanager --sdk_root=${ANDROID_SDK_ROOT} "platform-tools" "platforms;android-34" "build-tools;34.0.0" "ndk;${ANDROID_NDK_VERSION}" "cmdline-tools;latest"

# Set project directory
ARG PROJECT_NAME=qt-android
ENV PROJECT_DIR=/workspace/QtAndroidApp

# Copy project files into workspace
COPY . ${PROJECT_DIR}

# Set working directory
WORKDIR ${PROJECT_DIR}

# Create output directory
RUN mkdir -p /workspace/output/libs/arm64-v8a/

# Build Qt Android project and generate APK
RUN if [ -f "${PROJECT_DIR}/${PROJECT_NAME}.pro" ]; then \
        echo "=== Running QMake ===" && \
        /opt/qt/${QT_VERSION}/android_arm64_v8a/bin/qmake ${PROJECT_DIR}/${PROJECT_NAME}.pro && \
        echo "=== Running Make ===" && \
        make && \
        make install INSTALL_ROOT=/workspace/output && \
        echo "=== Checking installed files ===" && \
        find /workspace/output -type f && \
        echo "=== Checking all files ===" && \
        find /workspace -name "*.so" && \
        echo "=== Moving built files ===" && \
        mkdir -p /workspace/output/libs/arm64-v8a && \
        if [ ! -f "/workspace/output/libs/arm64-v8a/libQtAndroidApp_arm64-v8a.so" ]; then \
            cp -r /workspace/output/libs/arm64-v8a/libQtAndroidApp_arm64-v8a.so /workspace/output/libs/arm64-v8a/; \
        fi \
    fi


# Check if androiddeployqt6 exists, otherwise use androiddeployqt
RUN if [ ! -f "/opt/qt/${QT_VERSION}/android_arm64_v8a/bin/androiddeployqt6" ]; then \
        ln -s androiddeployqt /opt/qt/${QT_VERSION}/android_arm64_v8a/bin/androiddeployqt6; \
    fi

# Run androiddeployqt to generate APK
RUN if [ -f "${PROJECT_DIR}/android-${PROJECT_NAME}-deployment-settings.json" ]; then \
        echo "=== Running androiddeployqt ===" \
        && androiddeployqt6 --input ${PROJECT_DIR}/android-${PROJECT_NAME}-deployment-settings.json --output /workspace/output --android-platform android-34 --gradle; \
    fi

# Ensure APK is copied to output directory
RUN if [ -f "/workspace/output/build/outputs/apk/debug/app-debug.apk" ]; then \
        echo "=== Copying APK to output ===" \
        && cp /workspace/output/build/outputs/apk/debug/app-debug.apk /workspace/output/; \
    else \
        echo "=== APK NOT FOUND ==="; \
    fi

# Set default command
CMD ["bash"]
