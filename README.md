# FCM Keep Alive

Android app for auto-switching default IME:
- Screen off -> Gboard
- User present (unlock) -> WeChat IME

## Modes

### IME Switch (default)
Switches the default IME based on screen state: Gboard on screen-off, WeChat IME on unlock.

### Charging Keep Alive
Simulates AC charging state changes via Shizuku shell commands to keep FCM connections alive.

### Screen On FCM Heartbeat
Sends a single native FCM heartbeat broadcast each time the display turns on.

- **Trigger**: fires only on `ACTION_SCREEN_ON` (display physically turns on). Does **not** fire on unlock (`ACTION_USER_PRESENT`), screen-off, cast-setting state changes, or mirror-device state changes.
- **Implementation**: calls `sendBroadcast(Intent("com.google.android.intent.action.MCS_HEARTBEAT").setPackage("com.google.android.gms"))` via the normal Android app API.
- **No periodic broadcasts**: there is no timer, alarm, WorkManager job, or repeated scheduling — exactly one broadcast is dispatched per screen-on event.
- **No Shizuku required**: this mode uses only the standard Android `sendBroadcast` API and does not call Shizuku or execute shell commands.
- **Delivery note**: `sendBroadcast()` confirms dispatch initiation by the system. Whether Google Play services actually processes the heartbeat may be restricted by Android or Google Play services depending on device, build, or app signing. The log records the dispatch result (success or exception).

## Build
1. Open this folder in Android Studio (Jellyfish or newer).
2. Let Android Studio sync Gradle.
3. Build and install on your device.

## First-time setup on device
1. Enable both Gboard and WeChat keyboard in Android input settings.
2. In app, choose WeChat IME ID.
3. Gboard IME ID is fixed to `com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME`.

## Notes
- Auto switching default IME requires `WRITE_SECURE_SETTINGS`.
- Foreground notification is required for reliable background behavior.
- Service auto-restores after reboot.

## Whitelist
- Use the list from `adb shell dumpsys deviceidle whitelist` to set FCM apps to `No restrictions` in HyperOS battery saver settings.
- Use an Activity to launch the Stock Android battery settings and set FCM apps to `Unrestricted`

## Battery Saver
- No restrictions `adb shell dumpsys activity service com.miui.powerkeeper | findstr "scenario:8"`
- Apps in the list must be set to No restrictions

## adb
- `adb shell settings get secure default_input_method`
- `adb shell dumpsys activity services com.google.android.inputmethod.latin`
- `adb shell dumpsys battery reset`
- `adb shell dumpsys activity service com.google.android.gms/.gcm.GcmService | select -f 10`
