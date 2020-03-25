<br/><br/>
<p align="center">
  <img width="600" src="https://res.cloudinary.com/jesuhrz/image/upload/v1585169733/fastlane-and-firebase.png">
</p>
<br/>

# Distribution of apps with Fastlane and Firebase
Distribution of apps with firebase for apps made in IOS, Android or React Native

# Introduction
This repository was created with the intention of helping developers to improve and optimize your apps when they are distributed.

## Fastlane installation:

#### - Using RubyGems
```sh
sudo gem install fastlane -NV
```

#### - Using Homebrew
```sh
brew install fastlane
```

## Firebase CLI installation:
Follow the steps of the CLI firebase documentation: [Firebase CLI](https://firebase.google.com/docs/cli)

After installing Firebase CLI, run in the terminal:
```
firebase login
```

## IOS Setup:
Navigate to your IOS project folder and run:
```
fastlane init
```

Choose option 4 and follow the steps in the terminal


### Follow the next step:

#### - Add plugin
Run and follow the steps in the terminal:
```
fastlane add_plugin firebase_app_distribution
```

#### - Add .env file
Navigate to the fastlane folder of your project, create an .env file and copy the following:
```
SCHEME_NAME="" // Scheme name of your IOS project
BUNDLE_ID="" // Bundle ID of your IOS  project
DISTRIBUTION_PROFILE_FIREBASE="" // Profile name (InHouse or Ad Hoc)
APP_ID="" // App Id of your project in Firebase Console
GROUPS="" // Name of the groups created for distribution in Firebase Console
```

#### - Add new task
Add this new task in the Fastfile file
```
desc "Building your App in beta version"
lane :beta do |options|
    build(profile: options[:env])
    firebase_app_distribution(
      app: ENV['APP_ID'],
      groups: ENV["GROUPS"] || options[:groups],
      release_notes: options[:notes] || "",
    )
end

private_lane :build do |options|
    puts "+------------------------------------+".bold.blue
    puts "|-- Environment: #{options[:profile]} 🚀 --|".bold.blue
    puts "+------------------------------------+".bold.blue

    increment_build_number()

    scheme = ENV['SCHEME_NAME']
    method = (options[:profile] == 'distribution' ? "app-store" : "ad-hoc")
    profile = (options[:profile] == 'distribution' ? 
      ENV['DISTRIBUTION_PROFILE_APPSTORE'] : ENV['DISTRIBUTION_PROFILE_FIREBASE'])
    build_app(
      scheme: scheme,
      export_options: {
        method: method,
        provisioningProfiles: {
          ENV['BUNDLE_ID'] => profile
        }
      },
      include_bitcode: true,
      clean: true
    )
end
```

## Android Setup:
Navigate to your Android project folder, run and follow the steps in the terminal:
```
fastlane init
```

### Follow the next step:

#### - Add plugin
Run and follow the steps in the terminal:
```
fastlane add_plugin increment_version_code
```
```
fastlane add_plugin firebase_app_distribution
```

#### - Add .env file
Navigate to the fastlane folder of your project, create an .env file and copy the following:
```
APP_ID="" // App Id of your project in Firebase Console
GROUPS="" // Name of the groups created for distribution in Firebase Console
```

#### - Add new task
Add this new task in the Fastfile file
```
desc "Building your App in version beta"
lane :beta do |options|
  puts "+---------------------------------+".bold.blue
  puts "|-- Environment: #{options[:env]} 🚀 --|".bold.blue
  puts "+---------------------------------+".bold.blue

  increment_version_code(
    gradle_file_path: "app/build.gradle",
  )
  gradle(
    task: "clean assembleRelease"
  )
  firebase_app_distribution(
    app: ENV['APP_ID'],
    groups: ENV["GROUPS"] || options[:groups],
    release_notes: options[:notes] || "",
  )
end
```
## Finally we run our script:
```
fastlane beta env:development
```
