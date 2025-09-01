# Full cicd for releasing mobile app (both android + ios)

Here’s a complete, production-ready CI/CD blueprint for **building, signing, and releasing a mobile app to both Android & iOS** with **GitHub Actions + Fastlane**. I’ll assume a **Flutter** app (very similar for React Native—ask if you need that flavor).

---

# The Pipeline (high level)

* **PR checks (every PR)**
  Lint, type checks, unit tests, static analysis, build a debug APK/IPA for sanity.
* **Beta builds (push to `main`)**
  Build & sign → upload to **TestFlight** (iOS) and **Play Console Internal testing** (Android). Notify team.
* **Production release (tag `vX.Y.Z`)**
  Build & sign → upload to **App Store** & **Play Production** with staged rollout, using release notes from changelog.

---

# Secrets & credentials (one-time setup)

Add these to **GitHub → Settings → Secrets and variables → Actions**:

**Common**

* `FIREBASE_TOKEN` (if you also ship via Firebase App Distribution)
* `SENTRY_AUTH_TOKEN` (if using Sentry)

**Android**

* `ANDROID_KEYSTORE_BASE64` – base64 of your `upload.keystore`
* `ANDROID_KEYSTORE_PASSWORD`
* `ANDROID_KEY_ALIAS`
* `ANDROID_KEY_ALIAS_PASSWORD`
* `GOOGLE_PLAY_JSON` – base64 of Play Console **Service Account JSON**

**iOS**

* `APP_STORE_CONNECT_API_KEY_ID`
* `APP_STORE_CONNECT_API_ISSUER_ID`
* `APP_STORE_CONNECT_API_KEY` – base64 of the `.p8` key
* (Optional) `MATCH_PASSWORD` if using `match` (shared certs in a private repo)

> Store files as base64 and decode during jobs to avoid writing plaintext.

---

# Versioning & build numbers

* **Semantic Versioning** for marketing version (`1.4.2`).
* **Build number** auto-incremented per CI run:

  * iOS `CFBundleVersion = $GITHUB_RUN_NUMBER`
  * Android `versionCode = YYYYMMDD##` (date + short run number) or just `$GITHUB_RUN_NUMBER`.

You’ll set these in Fastlane before build.

---

# Fastlane setup

Install Fastlane in each native folder:

```bash
cd ios && fastlane init
cd ../android && fastlane init
```

### `ios/Fastfile`

```ruby
default_platform(:ios)

platform :ios do
  desc "Beta to TestFlight"
  lane :beta do
    # Use App Store Connect API Key (set via env vars)
    api_key = app_store_connect_api_key(
      key_id: ENV["APP_STORE_CONNECT_API_KEY_ID"],
      issuer_id: ENV["APP_STORE_CONNECT_API_ISSUER_ID"],
      key_content: Base64.decode64(ENV["APP_STORE_CONNECT_API_KEY"]),
      in_house: false
    )

    # Bump build number from CI
    build_num = ENV["GITHUB_RUN_NUMBER"] || Time.now.strftime("%Y%m%d%H%M")
    increment_build_number(build_number: build_num)

    # If Flutter: build via flutter build ios --no-codesign and then gym with export
    sh("flutter build ipa --release --no-codesign")

    # Automatically manage signing (or integrate match if you prefer)
    # gym will pick the .ipa from Flutter build if you point to it; otherwise:
    gym(
      scheme: "Runner",
      export_method: "app-store"
    )

    upload_to_testflight(
      api_key: api_key,
      skip_waiting_for_build_processing: true,
      distribute_external: true,
      groups: ["Internal"]
    )
  end

  desc "Release to App Store"
  lane :release do
    api_key = app_store_connect_api_key(
      key_id: ENV["APP_STORE_CONNECT_API_KEY_ID"],
      issuer_id: ENV["APP_STORE_CONNECT_API_ISSUER_ID"],
      key_content: Base64.decode64(ENV["APP_STORE_CONNECT_API_KEY"]),
      in_house: false
    )

    build_num = ENV["GITHUB_RUN_NUMBER"] || Time.now.strftime("%Y%m%d%H%M")
    increment_build_number(build_number: build_num)
    sh("flutter build ipa --release --no-codesign")
    gym(scheme: "Runner", export_method: "app-store")

    upload_to_app_store(api_key: api_key, submit_for_review: false)
    # Optional: automatically release
    # deliver(..., automatic_release: true)
  end
end
```

### `android/Fastfile`

```ruby
default_platform(:android)

platform :android do
  desc "Beta to Play Internal Testing"
  lane :beta do
    # Decode keystore
    sh("echo '${ANDROID_KEYSTORE_BASE64}' | base64 --decode > android/upload.keystore")

    # Set versionCode from CI
    version_code = (ENV["GITHUB_RUN_NUMBER"] || Time.now.strftime("%Y%m%d%H%M")).to_i
    # Update gradle files
    gradle(task: "setVersionCode", properties: {versionCode: version_code}) rescue nil

    sh("flutter build appbundle --release")

    upload_to_play_store(
      json_key_data: Base64.decode64(ENV["GOOGLE_PLAY_JSON"]),
      track: "internal",
      aab: "build/app/outputs/bundle/release/app-release.aab"
    )
  end

  desc "Release to Play Production"
  lane :release do
    sh("echo '${ANDROID_KEYSTORE_BASE64}' | base64 --decode > android/upload.keystore")
    version_code = (ENV["GITHUB_RUN_NUMBER"] || Time.now.strftime("%Y%m%d%H%M")).to_i
    gradle(task: "setVersionCode", properties: {versionCode: version_code}) rescue nil

    sh("flutter build appbundle --release")

    upload_to_play_store(
      json_key_data: Base64.decode64(ENV["GOOGLE_PLAY_JSON"]),
      track: "production",
      rollout: 0.1, # 10% staged rollout
      aab: "build/app/outputs/bundle/release/app-release.aab"
    )
  end
end
```

> For the `gradle(task: "setVersionCode")` call, you can add a small Gradle task in `android/app/build.gradle` to set the value, or directly edit `local.properties` / `build.gradle` with a script. If you prefer, skip that and use a single source of truth via `pubspec.yaml` + a prebuild script.

---

# GitHub Actions workflows

Create three workflows in `.github/workflows/`:

### 1) `ci.yml` – PR checks

```yaml
name: CI
on:
  pull_request:
    branches: [ "**" ]

jobs:
  test-and-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "stable"

      - name: Dependencies
        run: flutter pub get

      - name: Analyze
        run: flutter analyze

      - name: Unit tests
        run: flutter test --coverage

      - name: Build debug (sanity)
        run: |
          flutter build apk --debug
```

### 2) `beta.yml` – push to `main` → TestFlight & Play Internal

```yaml
name: Beta
on:
  push:
    branches: [ "main" ]

jobs:
  android-beta:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "stable"
          cache: true

      - name: Decode Android keystore
        run: echo "$ANDROID_KEYSTORE_BASE64" | base64 --decode > android/upload.keystore
        env:
          ANDROID_KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}

      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.2"
          bundler-cache: true

      - name: Install Fastlane
        run: gem install fastlane

      - name: Flutter pub get
        run: flutter pub get

      - name: Android beta lane
        working-directory: android
        env:
          ANDROID_KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          ANDROID_KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          ANDROID_KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          ANDROID_KEY_ALIAS_PASSWORD: ${{ secrets.ANDROID_KEY_ALIAS_PASSWORD }}
          GOOGLE_PLAY_JSON: ${{ secrets.GOOGLE_PLAY_JSON }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
        run: fastlane beta

  ios-beta:
    # iOS builds require macOS
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "stable"
          cache: true

      - name: Xcode select
        run: sudo xcode-select -s /Applications/Xcode.app

      - name: Ruby / Fastlane
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.2"
          bundler-cache: true

      - name: Install Fastlane
        run: gem install fastlane

      - name: CocoaPods
        run: |
          cd ios
          pod install --repo-update

      - name: iOS beta lane
        working-directory: ios
        env:
          APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
          APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
        run: fastlane beta
```

### 3) `release.yml` – tag `v*` → production

```yaml
name: Release
on:
  push:
    tags:
      - "v*.*.*"

jobs:
  android-release:
    runs-on: ubuntu-latest
    environment: production   # require approval
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "stable"
          cache: true
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.2"
          bundler-cache: true
      - run: gem install fastlane

      - name: Android release lane
        working-directory: android
        env:
          ANDROID_KEYSTORE_BASE64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
          ANDROID_KEYSTORE_PASSWORD: ${{ secrets.ANDROID_KEYSTORE_PASSWORD }}
          ANDROID_KEY_ALIAS: ${{ secrets.ANDROID_KEY_ALIAS }}
          ANDROID_KEY_ALIAS_PASSWORD: ${{ secrets.ANDROID_KEY_ALIAS_PASSWORD }}
          GOOGLE_PLAY_JSON: ${{ secrets.GOOGLE_PLAY_JSON }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
        run: fastlane release

  ios-release:
    runs-on: macos-14
    environment: production   # require approval
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "stable"
          cache: true
      - name: Xcode select
        run: sudo xcode-select -s /Applications/Xcode.app
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.2"
          bundler-cache: true
      - run: gem install fastlane

      - name: CocoaPods
        run: |
          cd ios
          pod install --repo-update

      - name: iOS release lane
        working-directory: ios
        env:
          APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ID }}
          APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
          GITHUB_RUN_NUMBER: ${{ github.run_number }}
        run: fastlane release
```

---

# App signing

**Android**
`android/app/build.gradle`:

```gradle
def keystoreProperties = new Properties()
def keystoreFile = rootProject.file("upload.keystore")
if (keystoreFile.exists()) {
    keystoreProperties['storeFile'] = keystoreFile.absolutePath
    keystoreProperties['storePassword'] = System.getenv("ANDROID_KEYSTORE_PASSWORD")
    keystoreProperties['keyAlias'] = System.getenv("ANDROID_KEY_ALIAS")
    keystoreProperties['keyPassword'] = System.getenv("ANDROID_KEY_ALIAS_PASSWORD")
}

android {
  // ...
  signingConfigs {
    release {
      if (keystoreFile.exists()) {
        storeFile file(keystoreProperties['storeFile'])
        storePassword keystoreProperties['storePassword']
        keyAlias keystoreProperties['keyAlias']
        keyPassword keystoreProperties['keyPassword']
      }
    }
  }
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled true
      shrinkResources true
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }
}
```

**iOS**
Prefer **App Store Connect API key + Xcode automatic signing** in CI, or use **fastlane match** to share certificates/profiles. If using match, add `MATCH_GIT_URL` secret and run `fastlane match appstore` in the lane.

---

# Changelog & release notes

* Generate notes from commits (Conventional Commits) with a tool like `conventional-changelog` or Fastlane’s `github_release`.
* On `release.yml`, create a GitHub Release from tag, attach artifacts (APK, AAB, IPA), and pass notes to stores.

Example step:

```yaml
- name: Create GitHub Release
  uses: softprops/action-gh-release@v2
  with:
    generate_release_notes: true
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

# Extras (nice to have)

* **Environments & approvals**: require manual approval for `production`.
* **Crash reporting**: attach dSYMs/Proguard mapping to **Sentry** in lanes.
* **Distribution to QA**: optionally ship to **Firebase App Distribution** on `beta`.
* **CodePush/OTA**: for React Native, add CodePush for instant JS updates (not applicable to pure Flutter Dart VM AOT).
* **Cache**: Flutter, CocoaPods, Gradle caches are already leveraged via actions and runners.
* **Artifacts**: upload APK/AAB/IPA from beta jobs for manual download by QA.

---

# How to use it

1. Add the Fastfiles and Gradle/iOS bits.
2. Add the three workflow YAMLs.
3. Add secrets.
4. Push a PR → see CI.
5. Merge to `main` → TestFlight + Play Internal.
6. Create a tag `v1.0.0` → Production rollout.
