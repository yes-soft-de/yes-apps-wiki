# iOS Continues Deployment 

To create an iOS app from flutter first add fastlane to the project, and then get the dependencies file by running a `flutter build ios` command. note that this doesn't have to finish, as soon as the `Podfile` is created it should be enough.

After that we do a configuration which we will discuss in a later document. After that we produce a app on TestFlight. and lastly we sign the app using fastlane match.

Unfortunately, most of the job is done using a single step. it can be divided. But we haven't done so yet. splitting it may save 0-45 minutes of build time depending on the project.

Finally we upload the built IPA to TestFlight.

![ios pipeline](C:\Users\moham\Documents\Servers\symfony-app-skeleton.wiki\pipeline-ios.png)

