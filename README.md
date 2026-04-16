# Foros Cuba - Android

Native Android wrapper for [fcuba.net](https://fcuba.net/), built on top of the [mgks/Android-SmartWebView](https://github.com/mgks/Android-SmartWebView) template (v8, MIT-licensed).

Package id: `net.fcuba.app`.

## Local build

Prerequisites:

- JDK 17 (Temurin recommended)
- Android SDK with platform `android-36`
- About 8 GB of free disk space for the Gradle cache

```bash
./gradlew assembleDebug
```

The debug APK lands at `app/build/outputs/apk/debug/app-debug.apk`.

## GitHub Actions CI

`.github/workflows/build.yml` builds on every push and pull request to `main`:

- Always produces a debug APK artifact (`fcuba-debug-<sha>`).
- On pushes to `main` (not PRs), additionally builds a signed release APK (`fcuba-release-<sha>`) using AGP injected signing.

For signed release builds, add these four repository secrets (`Settings -> Secrets and variables -> Actions`):

| Secret | Value |
|---|---|
| `KEYSTORE_BASE64` | Output of `base64 -w 0 keystore.jks` |
| `KEYSTORE_PASSWORD` | Keystore password |
| `KEY_ALIAS` | Key alias inside the keystore |
| `KEY_PASSWORD` | Key password |

Generate a keystore if you don't have one:

```bash
keytool -genkey -v -keystore keystore.jks -keyalg RSA -keysize 2048 \
  -validity 10000 -alias fcuba
```

## Firebase / FCM setup

The Firebase Gradle plugin is applied **only when `app/google-services.json` exists** (see `app/build.gradle:101`). Without the file, the app builds and runs normally - push notifications are simply disabled.

To enable FCM:

1. Firebase Console -> Project settings -> Add Android app.
2. Package name: `net.fcuba.app`.
3. Download `google-services.json` and drop it at `app/google-services.json`.
4. Rebuild.

The file is in `.gitignore` - do not commit it.

## Customization

| What | Where |
|---|---|
| URL, feature flags, plugins, version | `app/src/main/assets/swv.properties` |
| App name | `app/src/main/res/values/strings.xml` (`app_name`) |
| Brand colors | `app/src/main/res/values/colors.xml` (+ `values-night/` for dark mode) |
| Deep-link host | `app/src/main/AndroidManifest.xml` (MainActivity intent-filter) |
| Offline page | `app/src/main/assets/web/offline.html` |

### TODO: Launcher icon and splash logo

Still using the upstream Smart WebView placeholder artwork. Replace with Foros Cuba branding at:

- `app/src/main/res/mipmap-*/ic_launcher.*` (all densities: mdpi, hdpi, xhdpi, xxhdpi, xxxhdpi, plus adaptive `ic_launcher_foreground`/`_background`)
- `app/src/main/res/drawable/splash_logo.*` (if present)
- `app/src/main/assets/web/swv_splash_white.png` (offline page logo)

Use Android Studio's Image Asset Studio (`File -> New -> Image Asset`) with a 512x512 source PNG to regenerate all densities in one go.

## Deep linking / App Links

The manifest declares:

```xml
<intent-filter android:autoVerify="true" android:label="@string/app_name">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="fcuba.net" />
    <data android:scheme="http" android:host="fcuba.net" />
</intent-filter>
```

For links to open in the app **without** the chooser dialog, publish a Digital Asset Links file at `https://fcuba.net/.well-known/assetlinks.json` containing the app's SHA-256 signing fingerprint:

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "net.fcuba.app",
    "sha256_cert_fingerprints": ["AA:BB:CC:...the real fingerprint..."]
  }
}]
```

Extract the fingerprint from your keystore:

```bash
keytool -list -v -keystore keystore.jks -alias fcuba | grep SHA256
```

Validate the file with [Google's Statement List Generator and Tester](https://developers.google.com/digital-asset-links/tools/generator).

## Syncing from upstream

The fork retains the upstream commit history. Pull new Smart WebView releases with:

```bash
git fetch upstream --tags
git merge upstream/master   # or a specific tag: git merge 8.1.0
```

Expect conflicts on the files listed in the Customization table above - resolve by keeping our fcuba.net values.

## License

MIT, inherited from the upstream template. See `LICENSE`.
