# Android Deployment Pipeline Action

first we will see the action and then we shall explain the details of it.

```yaml
name: Android Deploy

on:
  push:
    branches: 
      - 'dev*'
      - 'release-v*'
  pull_request:
    branches:
      - 'dev*'
      - 'release-v*'
jobs:
  deploy-android:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - uses: actions/setup-ruby@v1
        with:
          ruby-version: '2.6'

      - name: Setup JDK
        uses: actions/setup-java@v1
        with:
          java-version: "12.x"

      - name: Setup flutter
        uses: subosito/flutter-action@v1
        with:
          flutter-version: "1.22.4"
          channel: "stable"

      - name: Install NDK
        run: echo "y" | sudo /usr/local/lib/android/sdk/tools/bin/sdkmanager --install "ndk;21.0.6113669" --sdk_root=${ANDROID_SDK_ROOT}

      - name: Configure Keystore
        run: |
          cd ./flutter/android
          echo "$ANDROID_KEYSTORE_FILE" > key.jks.b64
          base64 -d -i key.jks.b64 > app/key.jks
          echo "storeFile=key.jks" > key.properties
          echo "keyAlias=$KEYSTORE_KEY_ALIAS" >> key.properties
          echo "storePassword=$KEYSTORE_STORE_PASSWORD" >> key.properties
          echo "keyPassword=$KEYSTORE_KEY_PASSWORD" >> key.properties
          ls
          
        env:
          ANDROID_KEYSTORE_FILE: ${{ secrets.ANDROID_KEYSTORE_FILE }}
          KEYSTORE_KEY_ALIAS: ${{ secrets.KEYSTORE_KEY_ALIAS }}
          KEYSTORE_KEY_PASSWORD: ${{ secrets.KEYSTORE_KEY_PASSWORD }}
          KEYSTORE_STORE_PASSWORD: ${{ secrets.KEYSTORE_STORE_PASSWORD }}

      - name: Firebase Key
        run: |
          cd ./flutter/android
          echo "$FIREBASE_CONFIG" > firebase.json.b64
          base64 -d -i firebase.json.b64 > app/google-services.json
        env:
          FIREBASE_CONFIG: ${{secrets.FIREBASE_CONFIG}}

      # - name: Upgrade Flutter Version
      #   run: |
      #     flutter downgrade 1.22.5

      - name: Get Flutter Packages tools
        run: |
          cd ./flutter
          flutter pub upgrade
          flutter packages pub run build_runner build --delete-conflicting-outputs

      - name: Install bundle
        run: |
          cd ./flutter/android
          gem install bundler
          bundle update --bundler
          bundle config path vendor/bundle
          bundle install --jobs 4 --retry 3

      - name: Distribute app to Alpha track 🚀
        run: |
          cd flutter/android
          bundle exec fastlane alpha

      - name: Upload artifact to Github
        uses: actions/upload-artifact@v1
        with:
          name: release-apk
          path: flutter/build/app/outputs/flutter-apk/app-release.apk

      - name: Create release and upload apk
        uses: underwindfall/create-release-with-debugapk@v2.0.0
        env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.run_number }}
          asset_path: flutter/build/app/outputs/flutter-apk/app-release.apk
          asset_name: release.apk
          asset_content_type: application/zip

```

## Name

This is the name of the pipeline. in order to filter it in GitHub later.

## On

this refers to when the pipeline should start. in this case it is fired when any branch starting with `release` tag is updated, this includes `release-v1.0` or `release-v1.2.5` and the likes.

But, the action can also starts when we create a pull request into a another branch. for example in this workfile, when we create a pull request to `dev-mobile` the action fires and it starts building.

A good use of this might be to release a debug version when we create a pull request to the dev branch -- in our case `dev-mobile` -- and another workfile when we actually merge into the dev branch -- a merge is considered a push -- and when we merge into a master we can send an APK to the internal test in Google Play.



## Jobs

This is a series of steps the defines how to run the action.

Note, we can actually create a pipeline which will deploy both iOS and android in the same file using multiple jobs. but we don't recommend it.

1. `- uses: actions/checkout@v2`  fetch the files from the repo
2. `- uses: actions/setup-ruby@v1` this is important for `fastlane` since it's written in ruby.
3. `- name: Setup JDK`, for gradle and android.
4. `- name: Setup flutter` an action for fetching flutter, note that the latest supported is 1.22.4, if we want to upgrade (and we do), we shall update manully.
5. `- name: Install NDK` this is for native binding with flutter project. not critical for small projects, but in our skeleton case we need it. and we need it in this particular version. do not update until critical.
6. `- name: Configure Keystore` this is used to sign the app, it's important for other people to be able to install it, and it reduces the size dramatically. SECRETS:
   1. ANDROID_KEYSTORE_FILE: Base64 of the actual android keystore.
   2. KEYSTORE_KEY_ALIAS: the android key alias, usually `key`
   3. KEYSTORE_STORE_PASSWORD: the password used when creating the store file.
   4. KEYSTORE_KEY_PASSWORD: the password used when adding the key to the store file.
7. `Firebase Key`: google-services files should not be uploaded to github. and this is how we add them to the project. SECRETS:
   1. FIREBASE_CONFIG: base64 representation of the Firebase Key.
8. `- name: Upgrade Flutter Version`: this is used to upgrade flutter. to do so, we first upgrade to the latest version and then we downgrade to 1.22.5. An alternative to this would be to fork the repo that downloads flutter and upgrade it.
9. `- name: Get Flutter Packages tools` This is used for 2 things. first as a packages fetcher, and second to generate the missing code to be used for dependency injection. and this is what `flutter packages pub run build_runner build --delete-conflicting-outputs` actually does. this is good because it makes the code review process so much easier.
10. `- name: Install bundle`: for fastlane. 
11. `- name: Distribute app to Alpha track 🚀` building the actual APK. we discuss how this build command actually works and how to create it in the android-deploy document
12. `- name: Upload artifact to Github` uploading to the artifacts folder. we discuss this in android-deploy document.

13. `- name: Create release and upload apk`: we discuss this in android-deploy document. NOTE: GITHUB_TOKEN already is set by Github, so don't add it to the project secret.



And that is it. the action file explained.