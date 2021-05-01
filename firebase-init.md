# Starting a new firebase project

We need 3 sections activated in firebase for pretty much any project.

## Authentication

1. Login to [Firebase Console](https://console.firebase.google.com)
2. Create a new project
3. Go to Authentication Section
4. Enable these methods:
   1. Google
   2. Email (With passwordless link)
   3. Phone
   4. Apple

## Firestore

1. Login to [Firebase Console](https://console.firebase.google.com)

2. Login into the existing project

3. Go to firestore section

4. Press "Create database"

5. Choose a production setting

6. For ME projects choose Europe Location

7. Go to "Rules"

8. Change the rules into:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /chat_rooms/{document=**} {
         allow read, write;
       }
       match /users/{userId} {
       	allow create: if request.auth != null;
         allow read, update, delete: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```

9. Publish the new rules

## Crashlytics

1. Login to [Firebase Console](https://console.firebase.google.com)
2. Login into the existing project
3. Go to crashlytics section
4. press "enable crashlytics"
5. This process should be repeated for every single app connected to Firebase

## Adding it to android

1. Login to [Firebase Console](https://console.firebase.google.com)
2. Login into the existing project
3. Go to applications section
4. download `Google-Services.json`
5. add the class path `classpath 'com.google.gms:google-services:4.3.3'` to `android/build.gradle`
6. add the plugin to `android/app/build.gradle` as `apply plugin: 'com.google.gms.google-services'`

## Adding it to iOS

refer to [Prepairing iOS Projects](./prepaire-ios.md)
