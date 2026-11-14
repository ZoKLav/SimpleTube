# SimpleTube

SimpleTube is a modified version of the original YouTube for Android TV APK, built to reduce advertising and telemetry while keeping the original YouTube TV interface, Cobalt engine, and video player.

It exists because watching a video shouldn’t mean repeatedly sitting through advertising. The goal is to preserve the familiar YouTube experience while removing as much of the advertising infrastructure as practical.

This is an APK-level modification of YouTube TV **7.28.301**, rather than a replacement client built from scratch.

## Technical changes

### Application identity

- Changed the Android application package to `com.zoeykl.simpletube`.
- Changed all 82 application-label variants to **SimpleTube**.
- Re-signed the modified APK with a dedicated signing key, reused across subsequent builds.

SimpleTube has its own application identity and signing certificate. Google services may treat it differently from the official YouTube application.

### Single-APK ARMv7 packaging

The original installation used a base APK and a separate `armeabi-v7a` configuration split. SimpleTube combines them into one installable APK.

- Merged all six native libraries from the matching ARMv7 split, including the Cobalt playback library, `libchrobalt.so`.
- Removed the manifest requirements for a separate split installation.
- Kept the native libraries uncompressed and correctly aligned.
- Preserved the original assets and resources, apart from intentional branding and configuration edits.

The current build targets **32-bit ARMv7 Android TV devices**. Other architectures are outside the current scope.

### Advertising identifiers and telemetry

- Disabled the advertising-ID bridge used to obtain the device’s advertising identifier.
- Disabled the identified Clearcut telemetry upload path.
- Reduced additional telemetry in the patched Java layer.
- Added blocking rules for known advertising domains.

These changes do not establish that every tracking mechanism has been removed. The app still uses the original native engine and remotely loaded YouTube TV interface.

Domain blocking alone did **not** stop video ads during testing, which led to the page-level filtering added in r3.

### Google Play services error handling

Changing the package and signing certificate exposed a startup failure involving Google Play services package verification.

The affected connection path was patched so that the rejection reports a failed service connection through the existing callback instead of allowing the exception to terminate the app’s background thread.

This changes how SimpleTube handles the rejection; it does not make Google accept its package or certificate as the official application.

The existing DroidGuard authentication bridge using `kimono_auth` was retained because it is separate from advertising and may be involved in playback authentication.

### Page-level ad filtering

Revision 3 adds a JavaScript filter through the browser’s existing navigation path.

The filter targets ad scheduling data, including:

- `playerAds`
- `adPlacements`

The purpose is to remove those instructions before the TV interface passes them to the player, while preserving ordinary video data, captions, and unrelated responses.

The engine’s straightforward JavaScript-evaluation API was found to be restricted, so r3 uses the existing navigation path for the filter. The original native libraries and r2’s service-connection fix remain unchanged.

## Revision history

| Build | Changes |
|---|---|
| Initial base | Renamed the package and labels; patched advertising-ID collection, Clearcut uploads, telemetry, and ad-domain rules. Required the missing native split and could not run alone. |
| ARMv7 combined | Added all six matching native libraries and removed the separate split-install requirements. |
| r2 | Handled the Google Play services verification exception through the connection-failure callback. Sign-in and video playback were subsequently confirmed on a TV. |
| r3 | Added page-level filtering of ad scheduling data. Initial signed-out playback testing was encouraging, but broader verification remains ongoing. |

## Verification and current limitations

Build checks confirmed the application identity, labels, APK signature, ZIP alignment, native-library packaging, and preservation of the native libraries.

Local filter tests confirmed that the targeted ad scheduling fields were removed while video data, captions, and ordinary responses were preserved. The filter recovered from the compiled r3 APK passed the same tests.

On-device testing confirmed sign-in and playback with r2, although ads still appeared. Initial r3 testing suggests the page filter is working during signed-out playback. Signed-in ad suppression and reliable blocking across different videos and ad formats remain unverified.

YouTube can change its remotely delivered interface and response formats independently of this APK. Those changes may require updates to the filter.

SimpleTube is an unofficial modification and is not affiliated with or endorsed by YouTube or Google.
