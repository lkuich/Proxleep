# Proxleep Android app module

This folder contains the actual Android application for Proxleep, a small utility that locks the screen when the proximity sensor is covered.

## Module summary

- Package: `com.proxleep.loren.proxleep`
- App name: `Proxleep`
- Minimum SDK: `20`
- Target SDK: `25`
- Main component: `ProximityService`
- Device Admin policy: `force-lock`

The app was designed for an older Android phone whose physical power button had stopped working. Instead of pressing the button, the user could cover the proximity sensor for a short moment and the app would turn the screen off.

## Source files

### `MainActivity.java`

The launcher activity. It does not display a real UI. When opened, it starts `ProximityService` and then calls `finish()`.

### `ProximityService.java`

The core of the app.

Responsibilities:

- Gets the default `Sensor.TYPE_PROXIMITY` sensor.
- Registers a `SensorEventListener` when the service starts.
- Tracks whether the sensor is currently covered.
- Starts a short timer when the sensor reads as covered.
- Calls `DevicePolicyManager.lockNow()` to lock the screen when Device Administrator access is active.
- Contains a wake-lock based branch for waking the screen when it is not interactive.

The lock action depends on this check:

```java
active = deviceManager.isAdminActive(compName);
```

If the app has not been enabled as a Device Administrator, Android will not allow `lockNow()` to work.

### `Timer.java`

A simple `Thread` that waits about two seconds before asking the service to toggle the screen. This delay helps avoid locking the phone from quick accidental proximity changes.

### `Startup.java`

A `BroadcastReceiver` for `Intent.ACTION_BOOT_COMPLETED`. It restarts `ProximityService` after the device boots.

### `Admin.java`

A minimal `DeviceAdminReceiver`. Android requires this receiver, plus the policy XML, before an app can use the `force-lock` Device Administrator policy.

## Resources

### `src/main/res/xml/policies.xml`

Declares the only Device Admin policy used by this app:

```xml
<force-lock />
```

This grants the app the ability to lock the device immediately through `DevicePolicyManager.lockNow()` after the user enables Device Administrator access.

### `src/main/res/layout/activity_main.xml`

An empty layout. The activity exists only as an entry point for starting the service.

### `src/main/res/values/strings.xml`

Defines the app name and default generated strings from the original Android Studio project.

## Manifest entries

`src/main/AndroidManifest.xml` declares:

- `WAKE_LOCK` for wake-lock usage.
- `RECEIVE_BOOT_COMPLETED` so the service can start after reboot.
- `MainActivity` as the launcher activity.
- `Admin` as a Device Administrator receiver.
- `Startup` as a boot receiver.
- `ProximityService` as the background service.

## Build notes

This module uses the legacy Gradle dependency style:

```gradle
compile 'com.android.support:appcompat-v7:25.1.0'
```

Modern Android Gradle Plugin versions use `implementation` instead of `compile`, and modern Android releases have stricter background-service limits. Treat this app as a legacy Android project unless you plan to modernize it.

To build from the repository root with a compatible SDK/toolchain:

```bash
./gradlew assembleDebug
```

## Behavior in one sentence

Open Proxleep once, grant Device Administrator access, then cover the phone's proximity sensor for about two seconds to lock the screen without using the power button.
