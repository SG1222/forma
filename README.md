# Forma

Forma is a native Android photo journal that works completely offline. Log is a private photo library, Compare is a two-image workspace, and Streaks is a Monday-first weekly record.

There are no accounts, cloud services, analytics, advertising, or Internet permission. The app's only permission is the camera, plus the biometric permissions used by the optional app lock.

**Version:** 1.0 (version code 8) · **Application ID:** `com.localjournal.fitness` · **Android:** 10 (API 29) or later

## Features

- **Capture and import.** CameraX front and rear capture with supported flash controls, multiple photos per session, and imports from Android's system document picker. Originals are never modified.
- **Check-ins.** Dated entries with optional Front, Side, Back, or Other labels and a shared note. The date is recorded when you save, so importing an old photo does not backdate its check-in.
- **Log.** A three-column, month-grouped library of private thumbnails that keeps its scroll position.
- **Viewer.** Full-screen swipe, bounded pinch and pan, double-tap zoom, editing, and confirmed deletion.
- **Compare.** Side-by-side images with independent transforms, a clipped reveal slider with alignment controls, and Quick Compare.
- **Streaks.** Current and longest weekly streaks, total check-ins, and a calendar history. Streaks are derived from saved local dates.
- **Privacy controls.** An optional native biometric or device-credential lock (off by default), and System, Light, or Dark themes.
- **Export and deletion.** Explicit export of individual photos to the gallery, and a confirmed "Delete all data" action.

## Download

| File | Variant | Signature | Notes |
|---|---|---|---|
| `Forma-v1.0-debug.apk` | Debug | Existing debug certificate | Debuggable. Use this to update an existing debug-signed install. |
| `Forma-v1.0-release.apk` | Release | Release certificate | R8 and resource shrinking, not debuggable. |
| `Forma-Source-v1.0.zip` | Source | n/a | Full Android project, extracts to `Forma/`. |

The two APKs are signed with different certificates. The release APK **cannot update a debug-signed installation in place**. Do not uninstall an installed Forma to work around a signature mismatch, because uninstalling deletes the private journal.

## Building from Source

Extract `Forma-Source-v1.0.zip` and work from its `Forma/` folder. Requirements:

- JDK 17
- Android SDK Platform 36, build-tools 35.0.0, and platform-tools
- Android Studio compatible with AGP 8.13.2, or the command-line tools

Set `ANDROID_HOME` or create an uncommitted `local.properties` containing `sdk.dir=/your/sdk/path`. The Gradle wrapper pins its distribution checksum. Initial dependency resolution needs Internet access; the installed app does not.

```sh
chmod +x gradlew
./gradlew :app:assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

For normal local testing, use the optimized variant, which runs R8 and resource shrinking and is non-debuggable. It is signed with the local debug key and is **not a production release**:

```sh
./gradlew :app:assembleLocalOptimized
adb install -r app/build/outputs/apk/localOptimized/app-localOptimized.apk
```

To produce a signed release, supply `FORMA_RELEASE_KEYSTORE`, `FORMA_RELEASE_STORE_PASSWORD`, `FORMA_RELEASE_KEY_ALIAS`, and `FORMA_RELEASE_KEY_PASSWORD` in the build environment, then run `./gradlew :app:assembleRelease`. The keystore must be PKCS12. Without all four variables the release output is unsigned and must not be distributed. Do not place credentials in Gradle properties or source archives.

## Testing

```sh
./gradlew :app:assembleDebug :app:assembleDebugAndroidTest :app:testDebugUnitTest :app:lintDebug :app:verifyOfflinePrivacy
```

- **Unit tests** (`app/src/test`) cover streak boundaries, comparison math, storage and failure recovery, session policy, and color contrast.
- **Instrumentation tests** (`app/src/androidTest`) cover camera, import, check-in, viewer, Compare gestures, export and deletion, fault recovery, both themes, reduced motion, a 1,000-photo library, and responsive layouts.
- **Verification host** (`verification/`) is a separate application with its own UID. It runs black-box checks against the real optimized APK, covering capture, persistence, comparison, editing, export, and deletion.
- **Device runners** (`tools/`) are destructive. They must be run against exactly one disposable English API 35 emulator, never a device with personal data. See `tools/verify.py` and `tools/verify_optimized.py`.

`verifyOfflinePrivacy` fails the build if the merged manifest gains a permission outside the approved set, or if backup or device-transfer exclusions are incomplete.

## Architecture

```text
app/src/main/java/com/localjournal/fitness/
  core/          design, navigation, security, storage, export
  data/          Room database, models, DataStore preferences, repositories
  features/      camera, checkin, log, compare, streaks, viewer, settings
app/schemas/     exported Room schema (version 1)
```

- Compose screens receive state and actions through `JournalViewModel`. Room flows and DataStore supply journal, appearance, and authentication state.
- Repository mutations are serialized. Staged originals, atomic draft metadata, and a durable deletion marker protect saved entries if a write fails partway.
- Thumbnails and comparison images are decoded at display size. Thumbnails never fall back to full-resolution originals, and Coil disk caching is disabled.
- App lock uses native Android authentication. A fresh process locks when the lock is enabled. Returning from the background within 30 seconds does not require re-authentication.
- Gallery exports go to `Pictures/Photo Journal/` through MediaStore.

## Privacy and Security

- The merged manifest requests only `CAMERA`, `USE_BIOMETRIC`, `USE_FINGERPRINT`, and an internal receiver permission. It has no `INTERNET`, storage, or media permission, and includes no network, analytics, or cloud SDK.
- Captures, thumbnails, drafts, and temporary originals stay in app-private storage.
- Android backup is disabled, with explicit exclusions for legacy, cloud, and device-transfer paths.
- Photos rely on the Android sandbox and device protections. There is no app-level or end-to-end encryption.
- Manual exports are public, and other apps may back them up. "Delete all data" removes the private journal but leaves completed gallery exports alone.
- Uninstalling the app or clearing its data removes the private journal.

## Known Limitations

- Verified only on Android API 35 ARM64 software emulation. API 29 and 36 runtime, physical and OEM cameras and biometrics, backup and transfer behavior, and HEIF/HEIC are not verified.
- Not certified for TalkBack, landscape or foldable layouts, or specific OEM compliance.
- English only, with portrait-first single-pane layouts.
- Frame timing measured on the emulator is variable and does not establish a production frame rate.
- Power loss and full-device storage exhaustion were not tested on hardware. Fault tests inject IO and ENOSPC errors instead.

## Project Documentation

The full build, architecture, maintenance, and verification guide is in `Forma/README.md` inside the source archive. Implementation status is in `IMPLEMENTATION_STATUS.md`.

## License

No project-wide license has been established. Do not assume a license grant.

Third-party notices are in `THIRD_PARTY_NOTICES.md`. Vendored icon vectors are Apache-2.0 (`LICENSES/Apache-2.0.txt`). The approved Forma artwork is supplied by the project owner.
