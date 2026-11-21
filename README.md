name: Build BTC Android Miner APK

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout BTC Android Miner
        uses: actions/checkout@v4
        with:
          repository: VirtusVerbis/BTC_Anrdoid_Miner
          path: miner

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3
        with:
          packages: 'platform-tools platforms;android-34 build-tools;34.0.0 ndk;25.1.8937393 cmake;3.22.1'
          accept-android-sdk-licenses: true

      - name: Install Vulkan shader compiler
        run: |
          sudo apt-get update
          sudo apt-get install -y glslang-tools
          glslangValidator --version

      - name: Build debug APK
        working-directory: miner
        env:
          ANDROID_SDK_ROOT: /usr/local/lib/android/sdk
          ANDROID_HOME: /usr/local/lib/android/sdk
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug --stacktrace --no-daemon

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: btc-android-miner-debug
          path: miner/app/build/outputs/apk/debug/app-debug.apk
          if-no-files-found: error
