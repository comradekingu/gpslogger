GPSLogger  [![githubactions](https://github.com/mendhak/gpslogger/workflows/Android%20CI/badge.svg)](https://github.com/mendhak/gpslogger/actions) [![pgp](assets/pgp.png)](https://keybase.io/mendhak) [![Crowdin](https://badges.crowdin.net/gpslogger-for-android/localized.svg)](https://crowdin.com/project/gpslogger-for-android)
=========

Log GPS info (GPX, KML, CSV, NMEA, Custom URL) and upload it (SFTP, OpenStreetMap, Dropbox, e-mail). \
This Android app aims to be as battery efficient as possible.

[Read about GPSLogger's features here](https://gpslogger.app/)

## Download

<a href="https://f-droid.org/packages/com.mendhak.gpslogger/">
    <img src="https://fdroid.gitlab.io/artwork/badge/get-it-on.png"
    alt="Get it on F-Droid"
    height="80">
</a>
<a href="https://github.com/mendhak/gpslogger/releases">
    <img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png"
    alt="Get it on Google Play"
    height="80">
</a>

Or download directly [from the releases](https://github.com/mendhak/gpslogger/releases).


## Contribute

You can help with [translations](https://hosted.weblate.org/engage/gpslogger/) on Hosted Weblate.
<a href="https://hosted.weblate.org/engage/gpslogger/">
<img src="https://hosted.weblate.org/widgets/gpslogger/-/horizontal-auto.svg" alt="Oversettelsesstatus" />
</a>

You can also submit [pull requests](https://help.github.com/articles/using-pull-requests) for bugfixes and new features.

Any help with UI or layouts is greatly appreciated!  


## License and policy

[Licensed GPL v2+](LICENSE.md) | [Third party licenses](assets/text/opensource.md) | [Privacy policy](assets/text/privacypolicy.md)



## Verifying

It's good practice to verify downloads.  In recent releases, a PGP signature and an SHA-256 checksum will accompany each `.apk`.

Import the public PGP key from [Keybase.io](https://keybase.io/mendhak) or just use `gpg --recv-key 6989CF77490369CFFDCBCD8995E7D75C76CBE9A9`

To verify the integrity and signature:

    $ gpg --verify ~/Downloads/gpslogger-71.apk.asc
    
To verify the checksum:
    
    $ sha256sum -c ~/Downloads/gpslogger-71.apk.SHA256


Setting up the code
=========


The project is based on the [Android build system](http://tools.android.com/tech-docs/new-build-system/user-guide) plugin for Gradle.
These instructions are for Ubuntu Linux with Android Studio, but for other OSes, it should be roughly similar. 

### Set up your Android Development Environment

Follow the instructions on the [Android Developer Website](http://developer.android.com/sdk/installing/index.html) to set up your computer for development.

Download and install [Android Studio](https://developer.android.com/studio/install#linux) (there's also a [snap](https://snapcraft.io/android-studio))


### Clone the GPSLogger repository

    git clone git://github.com/mendhak/gpslogger.git

### Get the Android SDK extra repositories

This project uses certain Android libraries, you can install them using Google's poorly implemented [`sdkmanager`](https://developer.android.com/studio/command-line/sdkmanager.html):

      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'tools'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'platform-tools'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'build-tools;26.0.2'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'platforms;android-27'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'platforms;android-25'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'extras;google;m2repository'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'extras;android;m2repository'
      echo y | $HOME/android-sdk/tools/bin/sdkmanager 'extras;google;google_play_services'


### Create local.properties

Create a file called `local.properties`, pointing at your Android SDK directory.

    cd gpslogger
    echo "sdk.dir=/home/mendhak/Programs/Android" > local.properties

### Import the project

Open up Android Studio and choose to import a project. \
Select the topmost `build.gradle` file under GPSLogger.

If you get an Import dialog, choose to *Import project from external model*

![import](assets/import_1.png)

On the next screen, choose the defaults and proceed (default Gradle wrapper)

![import](assets/import_2.jpg)

Give it a minute and Android Studio will configure the projects and download the various libraries.

### OpenStreetMap Setup (Optional)

Sign up for an [OpenStreetMap](https://openstreetmap.org) account and log in.

Click on 'OAuth settings'

Click on 'Register your application'

Fill in the form with these details:

![OAuth settings](assets/osm_oauth_settings.png)

After registering the app, you will receive a 'Consumer Key' and a 'Consumer Secret'.  
Place the keys in OSMJob.java and OpenStreetMapManager.java.   


### Dropbox Setup (Optional)

Sign up for a Dropbox.com account and log in.

Go to the [Dropbox Developers page](https://www.dropbox.com/developers/apps) and click 'Create an App'

Use these settings, but choose a unique name

![Dropbox settings](assets/dropbox_settings_create.png)

After creating the app, you will receive an app key and secret (the ones in the screenshot are fake).

![Dropbox settings](assets/dropbox_settings.png)

Place the keys in your `~/.gradle/gradle.properties` like this:


    GPSLOGGER_DROPBOX_APPKEY=abcdefgh
    GPSLOGGER_DROPBOX_APPSECRET=1234123456


Replace the Dropbox app key to your AndroidManifest.xml file

    <!-- Change this to be db- followed by your app key -->
    <data android:scheme="db-12341234"/>

Overview
======

GPSLogger is composed of a few main components;

![design](assets/gpslogger_architecture.png)

### Event Bus

The event bus is where all the cross communication happens. \
Various components raise their events on the event bus, \
and other parts of the application listen for those events. \
The most important one is when a location is obtained, \
it is placed on the event bus and consumed by many fragments.

### GPS Logging Service

GPSLoggingService is where all the work happens. \
This service talks to the location providers (network and satellite). \
It sets up timers and alarms for the next GPS point to be requested. \
It passes location info to the various loggers \
so that they can write files. \
It also invokes the auto-uploaders so that they may send their files to Dropbox, OSM, etc.

It also passes info to the event bus.

### GPS Main Activity

This is the main visible form in the app. \
It consists of several 'fragments' — the simple view, detailed view and big view. \
It takes care of the main screen, the menus and toolbars. \
The fragments listen to the event bus for location changes and display it in their own way.

### Session and AppSettings

Floating about are two other objects.

`Session` contains various pieces of info related to the current GPSLogger run,
such as current file name, the last known location, satellite count, and any other non-static info which
needed for the current operation of GPSLogger.

`AppSettings` is a representation of the user's preferences.

These objects are visible throughout the app and can be accessed directly by any class, service, activity or fragment.


## Working notes for F-Droid

Use the fdroidserver Docker image. Clone the F-Droid metadata repo and make changes to the com.mendhak.gpslogger.yml file. 

    git clone https://gitlab.com/fdroid/fdroiddata.git
    cd fdroiddata

    # https://f-droid.org/en/docs/Submitting_to_F-Droid_Quick_Start_Guide/
    # initialize the metadata repo
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master init -v
    
    # lint your metadata yml
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master lint com.mendhak.gpslogger -v
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master readmeta
    
    # see if the latest tag will get picked up. 
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master checkupdates --auto com.mendhak.gpslogger
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master rewritemeta com.mendhak.gpslogger

    # build
    docker run --rm -v /home/mendhak/Android/Sdk:/opt/android-sdk -v $(pwd):/repo -e ANDROID_HOME:/opt/android-sdk registry.gitlab.com/fdroid/docker-executable-fdroidserver:master build -v -l com.mendhak.gpslogger
    
