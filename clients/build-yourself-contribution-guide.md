---
description: How to build the app on your own system, or how to contribute to the project
---

# Build Yourself / Contribution Guide

We encourage all contributions to this project! This guide will help you get your development environment set up and show you how to contribute effectively.

{% hint style="info" %}
This guide is tailored towards the Android Studio IDE. You can also build in VSCode, but some tools may not exist or may work differently.
{% endhint %}

## Code of Conduct

* Write clean, well-commented code
* Follow [Dart documentation guidelines](https://dart.dev/guides/language/effective-dart)
* Format your code with `dart format ./ --line-length=120`
* Test your changes thoroughly before submitting
* Avoid making large formatting changes to files, unless that is the goal of your PR — it makes it easier for us to review your changes

## Development Environment Setup

### Prerequisites

1. **Install Git**: [Download](https://git-scm.com/downloads)
2. **Install Java JDK**: [Download](https://www.oracle.com/java/technologies/javase-downloads.html)
3. **Install Flutter Version Manager (FVM)** - Recommended

   FVM allows you to manage multiple Flutter versions easily:

   ```bash
   dart pub global activate fvm
   ```

   Then install and use the Flutter version specified in the project:

   ```bash
   fvm install
   fvm use
   ```

   Alternatively, install Flutter directly, following the guide for your OS all the way up to the "Test Drive" page: [Flutter installation guide](https://docs.flutter.dev/get-started/install)

4. **Install Android Studio**: [Download](https://developer.android.com/studio)
   * Install Flutter & Dart plugins via the Plugin Manager
   * Install Command Line Tools via SDK Manager
   * Configure an Android Virtual Device (AVD) via AVD Manager, or connect a physical Android device with USB debugging enabled
5. **Install Visual Studio Code** (Recommended): [Download](https://code.visualstudio.com/download)

   Install the following extensions:

   * Dart
   * Flutter
   * IntelliCode (optional but recommended)

### Android Device Setup

**Option 1: Android Virtual Device (AVD)**

* Open Android Studio → AVD Manager
* Create a new virtual device
* Choose a device definition (e.g., Pixel 6)
* Select a system image (API 30 or higher recommended)
* Finish setup and launch the emulator

**Option 2: Physical Android Device**

* Enable Developer Options on your device
* Enable USB Debugging
* Connect via USB
* Verify connection: `flutter devices`

## Repository Setup

### Forking and Cloning

1. Create a GitHub account if you don't have one
2. Fork the `bluebubbles-app` repository
3. Clone your forked repository:

   ```bash
   # HTTPS
   git clone https://github.com/YOUR_USERNAME/bluebubbles-app.git

   # SSH (recommended)
   git clone git@github.com:YOUR_USERNAME/bluebubbles-app.git
   ```

4. Set up the upstream remote:

   ```bash
   cd bluebubbles-app
   git remote add upstream git@github.com:BlueBubblesApp/bluebubbles-app.git
   ```

5. Fetch branches and pull the latest changes:

   ```bash
   git fetch --all
   git pull upstream master
   ```

{% hint style="info" %}
We build off of the `master` branch — always branch from, and open your pull requests against, `master`, **not** `development`.
{% endhint %}

### Installing Dependencies

```bash
# If using FVM
fvm flutter pub get

# If using global Flutter
flutter pub get
```

## Building the App

### Development Builds

Run the app in debug mode:

```bash
# If using FVM
fvm flutter run

# Standard Flutter
flutter run
```

### Production Builds

Build release APKs with flavor support:

```bash
# Beta flavor
flutter build apk --flavor=beta --release

# Production flavor
flutter build apk --flavor=prod --release

# Split per ABI (smaller file sizes)
flutter build apk --flavor=beta --release --split-per-abi
```

#### Flavors

The Android app is built using [Flutter build flavors](https://docs.flutter.dev/deployment/flavors), which let several variants of the app (different app names, icon colors, and application/package IDs) ship from the same codebase without conflicting with each other on a device. Flavors are defined in the `productFlavors` block of `android/app/build.gradle`:

* **prod**: Production builds for the Google Play Store (`com.bluebubbles.messaging`) - this is the default flavor
* **prodNoAa**: Identical to `prod`, but without Android Auto support
* **beta**: Beta testing builds with Firebase Test Lab integration
* **alpha**: Internal alpha testing builds
* **joel** / **tanay**: Personal developer builds used by maintainers, signed with the debug key so they build without any keystore setup

To build or run a specific flavor:

```bash
flutter run --flavor=prod
flutter build apk --flavor=beta --release
```

**Adding your own flavor**

If you'd like your own personal flavor (for example, to install a dev build alongside a production install without them overwriting each other), add a new entry to the `productFlavors` block in `android/app/build.gradle`:

```gradle
productFlavors {
    myflavor {
        dimension "app"
        resValue "string", "app_name_en", "BlueBubbles (My Flavor)"
        resValue "color", "ic_launcher_background", "#4c49de"
        resValue "string", "file_provider", "com.bluebubbles.messaging.myflavor.fileprovider"
        applicationId "com.bluebubbles.messaging.myflavor"
    }
}
```

Then give it a signing config in the `buildTypes.release` block:

```gradle
buildTypes {
    release {
        productFlavors.myflavor.signingConfig signingConfigs.debug // or signingConfigs.release
        // ...other flavors
    }
}
```

Assigning `signingConfigs.debug` lets the flavor build immediately using Flutter's auto-generated debug keystore. Only assign `signingConfigs.release` once you've set up your own keystore, as described below.

### Signing Keys (Keystore)

**Debug builds** — `flutter run`, or any flavor assigned `signingConfigs.debug` (like `joel` and `tanay`) — require no setup from you. Flutter automatically generates a debug keystore (`~/.android/debug.keystore`) the first time you build, and it's used automatically.

**Release builds** — `flutter build apk --release`, for flavors assigned `signingConfigs.release` (`prod`, `prodNoAa`, `beta`, `alpha`) — need a keystore you control:

1. Generate a keystore with the JDK's `keytool`:

   ```bash
   keytool -genkey -v -keystore ~/bluebubbles-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias bluebubbles
   ```

2. Create `android/key.properties` (this file is gitignored — never commit it):

   ```properties
   storePassword=<your keystore password>
   keyPassword=<your key password>
   keyAlias=bluebubbles
   storeFile=/absolute/path/to/bluebubbles-release-key.jks
   ```

`android/app/build.gradle` reads `key.properties` automatically and wires it into `signingConfigs.release` — no other changes are needed. If `key.properties` is missing, any flavor using `signingConfigs.release` will fail to build with a signing error. While testing locally, it's often easiest to just build one of the debug-signed flavors (`joel`/`tanay`) instead of setting up a keystore.

### Web

In the terminal window at the root of the project directory, run `flutter build web --web-renderer=canvaskit`. It will output the build files to `build/web` to be hosted on your server.

### Desktop

#### Windows Build

1. Install NuGet package manager
2. Go to Visual Studio Installer -> Modify Build Tools -> Individual Components and install the latest Windows 10 SDK
3.  Run the following commands:

    ```cmd
    flutter clean
    git reset
    flutter build windows
    ```

#### Linux Build

Under construction...

## Code Formatting

Always format your code before committing:

```bash
dart format ./ --line-length=120
```

This project uses a max line length of **120 characters**.

## Workflow: Picking an Issue

1. Check the [issues page](https://github.com/BlueBubblesApp/bluebubbles-app/issues)
2. Filter by labels:
   * `Difficulty: Easy`, `Difficulty: Medium`, `Difficulty: Hard`
   * `good first issue` - Great for newcomers
   * `bug` - Bug fixes
   * `enhancement` - New features
3. If working on something without an issue, create one first for tracking

{% hint style="info" %}
If you're new to Flutter development, look out for the `good-first-issue` or the `Difficulty: Easy` label on GitHub. These will be easier issues to help you start learning Flutter, without dealing with an overly-complex bug or feature.
{% endhint %}

## Workflow: Making Changes

1. **Create a feature branch off of `master`:**

   ```bash
   git checkout master
   git pull upstream master
   git checkout -b <your-name>/<feature|bug>/<short-description>
   # Example: git checkout -b john/feature/dark-mode-support
   ```

2. **Make your changes**
3. **Format your code:**

   ```bash
   dart format ./ --line-length=120
   ```

4. **Test thoroughly** on both emulator and physical device if possible. Make sure you don't commit any of your changes that comment out `onContentCommit`.

   {% hint style="warning" %}
   The client apps have a lot of variables that need to be tested. For example, if you're making a UI change, please make sure it looks good in all the default themes, and all the skins as well.

   If you wish to make a backend change, we suggest you consult with the main developers before writing code. This is so we can come up with a plan of attack and make sure we don't degrade existing functionality or create bugs.
   {% endhint %}

5. **Stage and commit:**

   ```bash
   git add <file>
   # or
   git add -A

   git commit -m "Clear, descriptive commit message"
   ```

6. **Push to your fork:**

   ```bash
   git push origin <your-branch-name>
   ```

## Workflow: Submitting a Pull Request

1. Go to your forked repository on GitHub
2. Click "Pull requests" → "New pull request"
3. Set the base repository to `BlueBubblesApp/bluebubbles-app` and the base branch to `master`
4. Include in your PR description:
   * **Problem**: What issue does this solve?
   * **Solution**: How did you fix it?
   * **Testing**: How did you test the changes?
   * **Screenshots**: If applicable, include before/after screenshots
5. Submit and wait for review!

## Additional Resources

* [Dart Documentation](https://dart.dev/guides)
* [Flutter Documentation](https://flutter.dev/docs)

## Questions?

Join our [Discord community](https://discord.gg/hbx7EhNFjp) for help and discussions!
