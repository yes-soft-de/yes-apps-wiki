# Fastlane Android

To start using fastlane in android, we need a couple of steps. in this document we shall discuss them and explain the fastfile in full details.

## Installing Fastlane

To do so, first we need to download ruby, the large one `ruby-dev` 

Then we download Bundler using `gem install bundler` 

we need to create a new `Gemfile`

in it we write

```ruby
source "https://rubygems.org"

gem "fastlane"
```

Now we install fastlane using

```
bundle install
```

Now we have fastlane installed. to configure it we start by creating a folder named `fastlane`. in this folder we create a file named `Fastfile` and inside we write:

```ruby
default_platform(:android)
platform :android do
  desc "Submit a new build to alpha Track on Play"
  lane :alpha do
    gradle(
      gradle_path: "/usr/bin/gradle",
      task: "assemble",
      build_type: 'Debug'
    )
    gradle(
      gradle_path: "/usr/bin/gradle",
      task: "assemble",
      build_type: 'Profile'
    )
    gradle(
      gradle_path: "/usr/bin/gradle",
      task: "assemble",
      build_type: 'Release'
    )
  end
end
```

Notice that all these pipes are gradle pipes. and that we repeated the build 3 times using debug, profile and release. this is because of a bug in `flutter build apk` where when it can't find the debug files it returns an error.

This might be fixed in future updates, I haven't tested them though!

In case we wanted build an `aab` or and `app bundle` we can use the appropriate pipeline using:

```ruby
default_platform(:android)
platform :android do
  desc "Submit a new build to alpha Track on Play"
  lane :alpha do
    gradle(
      gradle_path: "/usr/bin/gradle",
      task: "bundle",
      build_type: 'Release'
    )
  end
end
```

The error with APKs doesn't show here, so using this simple script is enough.

NOTE: we can use fastlane plugins like `supply`. but the app should be created manually before sending the app. and we need an additional values here.

## Appfile

in this simple config file named `appfile` located in the `fastlane` folder, we can use this script:

```ruby
json_key_file("play-store-credentials.json")
package_name("my.package.name")
```

Note that `play-store-credentials.json` can be supplied to the CI system from a GitHub secret and that this secret is actually a service account credential created from Google Cloud Console and granted access to the play publish console by the admin!

