# Proxleep

Proxleep is a tiny Android utility that turns a phone's screen off with the proximity sensor. I originally made it when the power button on my Nexus 5 broke: instead of pressing the button, I could cover the sensor near the earpiece and let the app lock the device for me.

The project is intentionally small and old-school. It is mostly a snapshot of an Android app from the Android 5/6 era, not a modern Play Store-ready application.

## What it does

- Starts a background service that listens to the device proximity sensor.
- Detects when something is covering the sensor, for example a hand, pocket, or flip cover.
- Waits briefly so accidental passes over the sensor do not immediately lock the phone.
- Uses Android's Device Administrator `force-lock` policy to turn the screen off.
- Starts itself again after the phone boots.

The app does not have much of a user interface. Opening it starts the service and closes the launch activity.

## How it works

The app is built around a few simple classes:

- `MainActivity` starts `ProximityService` and immediately finishes.
- `ProximityService` registers a `SensorEventListener` for `Sensor.TYPE_PROXIMITY`.
- When the proximity reading reports "near", the service starts a short `Timer` thread.
- If the sensor remains covered for about two seconds, the timer calls `toggleScreen()`.
- `toggleScreen()` uses `DevicePolicyManager.lockNow()` to lock the device when the screen is on.
- `Startup` listens for `BOOT_COMPLETED` and starts the service after reboot.
- `Admin` is the `DeviceAdminReceiver` required by Android before an app can call `lockNow()`.

Android requires Device Administrator access for apps that lock the screen programmatically. The policy declaration lives in `app/src/main/res/xml/policies.xml` and only requests the `force-lock` policy.

## Repository layout

```text
.
├── app/                         Android application module
│   ├── build.gradle             App build configuration
│   └── src/main/
│       ├── AndroidManifest.xml  Permissions, receivers, and service declarations
│       ├── java/...             Main app/service code
│       └── res/                 App resources and Device Admin policy XML
├── build.gradle                 Root Gradle build file
├── settings.gradle              Includes the `app` module
└── gradlew / gradlew.bat        Gradle wrapper scripts
```

## Building

This project uses an old Android Gradle Plugin and SDK configuration:

- Android Gradle Plugin: `2.2.3`
- Compile SDK: `25`
- Build Tools: `25.0.2`
- Minimum SDK: `20`
- Target SDK: `25`

If you have a compatible old Android SDK installed, try:

```bash
./gradlew assembleDebug
```

Modern Android Studio/Gradle versions may require upgrading the Gradle plugin, dependency declarations, SDK versions, and Android background-service behavior before the project builds cleanly.

## Installing and using

1. Build and install the debug APK on an Android device with a proximity sensor.
2. Open Proxleep once. The activity starts the background service and closes.
3. Grant Device Administrator permission if prompted. This permission is required so the app can lock the screen.
4. Cover the proximity sensor for roughly two seconds to turn the screen off.

On many phones the proximity sensor is near the top speaker/earpiece. If nothing happens, confirm that the device has a proximity sensor and that Device Administrator access is enabled for Proxleep.

## Important notes

- Because this is a legacy app, Android 8+ background-service restrictions may prevent it from behaving exactly as it did on older phones.
- Device Administrator permission must be disabled before uninstalling the app on many Android versions.
- The manifest includes boot startup support through `RECEIVE_BOOT_COMPLETED`.
- The app is not intended as a security product; it is a convenience workaround for hardware button problems.

## Why "Proxleep"?

The name is a mashup of "proximity" and "sleep": cover the proximity sensor, put the phone to sleep.
