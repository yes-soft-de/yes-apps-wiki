# iOS Action

First, make sure that you finish preparing the iOS project. I shall explain how using a separate document.

For now, we shall explain the action.

```yaml
name: iOS Deploy

on:
  push:
    branches: 
      - 'release-v*'
      - 'dev*'
  pull_request:
    branches: 
      - 'release-v*'
      - 'dev*'

jobs:
  deploy:
    runs-on: macos-latest
    steps:
    - uses: actions/checkout@v2

    - name: Setup JDK
      uses: actions/setup-java@v1
      with:
        java-version: "12.x"
    
    - name: Setup flutter
      uses: subosito/flutter-action@v1
      with:
        flutter-version: "1.22.4"
        channel: "stable"

    # - name: Upgrade Flutter Version
    #   run: |
    #     flutter downgrade 1.22.5

    - name: Select Xcode version
      run: sudo xcode-select -s '/Applications/Xcode_12.4.app/Contents/Developer'

    - name: Setup SSH Keys and known_hosts for fastlane match
      run: |
        SSH_PATH="$HOME/.ssh"
        mkdir -p "$SSH_PATH"
        touch "$SSH_PATH/known_hosts"
        echo "$PRIVATE_KEY" > "$SSH_PATH/id_rsa"
        chmod 700 "$SSH_PATH"
        ssh-keyscan github.com >> ~/.ssh/known_hosts
        chmod 600 "$SSH_PATH/known_hosts"
        chmod 600 "$SSH_PATH/id_rsa"
        eval $(ssh-agent)
        ssh-add "$SSH_PATH/id_rsa"
      env:
        PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Clean the build
      run: cd ./flutter && flutter clean

    - name: Bundle install
      run: cd ./flutter/ios && bundle update --bundler && bundle install

    - name: Install Flutter dependencies
      run: |
        cd flutter && flutter pub get
        flutter packages pub run build_runner build --delete-conflicting-outputs
        cd ./ios && pod install

    - name: Deploy to TestFlight
      run: |
        cd ./flutter/ios && bundle exec fastlane beta
      env:
        TEAM_ID: ${{ secrets.TEAM_ID }}
        ITC_TEAM_ID: ${{ secrets.ITC_TEAM_ID }}
        FASTLANE_USER: ${{ secrets.FASTLANE_USER }}
        FASTLANE_PASSWORD: ${{ secrets.FASTLANE_PASSWORD }}
        FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD: ${{ secrets.FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD }}
        FASTLANE_SESSION: ${{ secrets.FASTLANE_SESSION }}
        MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
        MATCH_KEYCHAIN_NAME: ${{ secrets.MATCH_KEYCHAIN_NAME }}
        MATCH_KEYCHAIN_PASSWORD: ${{ secrets.MATCH_KEYCHAIN_PASSWORD }}

```

Really, most of the action is secret fields, so let's get into it.

## name: iOS Deploy

The name of the action

## on

When to fire the action. 

## job

The actual actions:

### deploy

Name of the job. for now we only have one job, although it's possible to have more than one.

###     - uses: actions/checkout@v2

Pull the code files.

###     - name: Setup JDK

Add JDK to the build.

###     - name: Setup flutter

Add flutter to the build. note that the latest release of flutter supported by this plugin is 1.22.4. in theory 1.22.5 and 1.22.4 are basically the same and what is buildable in 1.22.4 is buildable 1.22.5 and vice versa.

### - name: Upgrade Flutter Version

This is where we specify the version we use in the action. If you want to use 1.22.5 or maybe 2.0.1 feature, just upgrade flutter and it will work. Also, if you wanted to use specifically 1.22.5 upgrade to the latest version and then downgrade to the specific version you like. it will take like a minute or two in build time.

###     - name: Select Xcode version

XCode version selection.

###     - name: Setup SSH Keys and known_hosts for fastlane match

This is really important. using this we add the <b>private</b> ssh key into the ssh client. this allows us to access the private repo that contains the certificates to sign the iOS app. note that this key should match the deploy key used in the private keys repository.

Note also, that this can be used as a bash script instead of the action file.

| Secret          | Objective                                                    |
| --------------- | ------------------------------------------------------------ |
| SSH_PRIVATE_KEY | the SSH private key pair to pull private match key repository. |



###     - name: Clean the build

Just to clean any previous build DI files. this is a backward compatibility step and it can be skipped if the skeleton with the appropriate `.gitignore` was used.

###    - name: Bundle install

This is a script to update bundler, which is essential for fastlane to use. Ruby is already installed in MacOS so you will not need to use it.

###     - name: Install Flutter dependencies

`flutter pub get` and installing the pod file. those files are huge and it can take up to 2 minutes for them to be installed. in the end a new file will be created in the CI which is a `Podfile.lock`. to speed things up we can generate this in the companies project. and it is really a more secure option to do so.

One major plus using a locally generated `Podfile.lock` is to use an already generated firebase and Firestore libs, which will decrees build times drastically.

###    - name: Deploy to TestFlight

Start the fastlane pipeline.

| Secret                                       | Objective                                                    |
| -------------------------------------------- | ------------------------------------------------------------ |
| TEAM_ID                                      | The team id. acquired from Apple Console                     |
| ITC_TEAM_ID                                  | the iTunes Connect Team ID from (https://appstoreconnect.apple.com/WebObjects/iTunesConnect.woa/ra/user/detail) |
| FASTLANE_USER                                | Account holder name.                                         |
| FASTLANE_PASSWORD                            | Account holder password.                                     |
| FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD | Generated Application Specific password, generated from the account console of Apple. |
| FASTLANE_SESSION                             | A session to use to upload the app to TestFlight. Generated from running `fastlane spaceauth` inside the `ios` directory of the project. |
| MATCH_PASSWORD                               | A password to decrypt the match files (described in Fastlane iOS document) |
| MATCH_KEYCHAIN_NAME                          | Keychain name, usually `login`. (described in Fastlane iOS document) |
| MATCH_KEYCHAIN_PASSWORD                      | Keychain decryption key. (described in Fastlane iOS document) |

That's all falks!

