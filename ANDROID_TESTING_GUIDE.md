# Android Plugin Compilation & Testing Guide

## Project Analysis

This is a Flutter plugin for **background location tracking** that:
- **Android**: Uses Google Play Services `FusedLocationProvider` API (with fallback to Android LocationManager)
- **iOS**: Uses CoreLocation framework
- Provides continuous location updates even when the app is in the background
- Uses foreground services on Android (required for background location)

### Project Structure
```
flutter_background_location/
├── lib/
│   └── background_location.dart          # Dart plugin interface
├── android/
│   ├── build.gradle                      # Plugin build configuration
│   └── src/main/kotlin/...               # Android native implementation
└── example/                              # Test app
    ├── lib/main.dart                     # Example Flutter app
    └── android/                          # Example Android app config
```

### Key Android Components
- **BackgroundLocationPlugin.kt**: Main plugin class (Flutter ↔ Android bridge)
- **BackgroundLocationService.kt**: Handles method channel communication
- **LocationUpdatesService.kt**: Foreground service for location updates
- **AndroidManifest.xml**: Declares permissions and services

---

## Prerequisites

Before compiling, ensure you have:

1. **Flutter SDK** installed (check: `flutter --version`)
   - Required: Flutter >=3.0.0, Dart SDK >=3.1.0
   
2. **Android Studio** or Android SDK installed
   - Android SDK Platform 34 (compileSdkVersion)
   - Minimum SDK: 18 (Android 4.3+)
   
3. **Java Development Kit (JDK)** 8 or higher
   - Check: `java -version`
   
4. **Physical Android device** or **Android Emulator**
   - Location services require a device/emulator (not just compilation)
   - Enable location services on emulator: Settings → Location → ON

---

## Step-by-Step Compilation & Testing

### Step 1: Navigate to Project Root
```bash
cd "/Users/ashfaqbinrahman/Freelance Works/school-app/forks/flutter_background_location"
```

### Step 2: Get Flutter Dependencies
```bash
# Get dependencies for the plugin itself
flutter pub get

# Navigate to example app and get its dependencies
cd example
flutter pub get
cd ..
```

### Step 3: Verify Android Configuration

Check that the example app's `AndroidManifest.xml` has required permissions:
- ✅ `ACCESS_COARSE_LOCATION`
- ✅ `ACCESS_FINE_LOCATION`
- ✅ `FOREGROUND_SERVICE`
- ✅ `FOREGROUND_SERVICE_LOCATION` (for Android 10+)

The example app should already have these configured.

### Step 4: Verify Flutter Setup

Check Flutter can detect your setup:
```bash
flutter doctor
```

Fix any issues shown (especially Android toolchain). If Android licenses are missing:
```bash
flutter doctor --android-licenses
```

**Note**: For Flutter plugins, you don't need to run `gradlew` directly. Flutter handles the build process automatically.

### Step 5: Connect Android Device/Emulator

**Option A: Physical Device**
1. Enable Developer Options on your Android device
2. Enable USB Debugging
3. Connect via USB
4. Verify: `flutter devices` should show your device

**Option B: Android Emulator**
1. Open Android Studio → AVD Manager
2. Create/Start an emulator (API 18+)
3. Verify: `flutter devices` should show your emulator

**Important**: Enable location services on the device/emulator!

### Step 6: Build and Run the Example App

```bash
# From project root
cd example

# Run on connected device/emulator
flutter run
```

Or build APK first:
```bash
flutter build apk --debug
# APK will be at: example/build/app/outputs/flutter-apk/app-debug.apk
```

### Step 7: Test the Plugin Functionality

Once the app launches:

1. **Grant Location Permissions**
   - The app will request location permissions
   - Grant "Allow all the time" for background location

2. **Start Location Service**
   - Tap "Start Location Service" button
   - You should see a foreground notification (Android)
   - Location data should start updating in the UI

3. **Verify Location Updates**
   - Check that latitude, longitude, accuracy, etc. are updating
   - Check console logs for location data

4. **Test Background Behavior**
   - Put app in background (press home button)
   - Location updates should continue
   - Check notification is still showing

5. **Stop Service**
   - Tap "Stop Location Service"
   - Updates should stop

---

## Troubleshooting

### Build Errors

**Error: "SDK location not found"**
```bash
# Create local.properties in example/android/
echo "sdk.dir=/path/to/your/android/sdk" > example/android/local.properties
```

**Error: "Gradle sync failed"**
```bash
cd example/android
./gradlew clean
cd ../..
flutter clean
flutter pub get
```

**Error: "Kotlin version mismatch"**
- Check `android/build.gradle` - Kotlin version is `1.7.10`
- Ensure Gradle plugin version is compatible

### Runtime Errors

**"Permission denied"**
- Check AndroidManifest.xml has all required permissions
- Ensure you granted runtime permissions in app settings
- For Android 10+: Grant "Allow all the time" not just "While using app"

**"Service not starting"**
- Check notification is configured (required for foreground service)
- Verify `LocationUpdatesService` is declared in AndroidManifest.xml
- Check logs: `flutter logs` or `adb logcat`

**"No location updates"**
- Ensure location services are enabled on device
- Check device has GPS signal (use Google Maps to verify)
- For emulator: Set location in Extended Controls → Location

### Debug Commands

```bash
# View Flutter logs
flutter logs

# View Android logs
adb logcat | grep -i "background_location\|location"

# Check connected devices
flutter devices

# Clean build
flutter clean
flutter pub get
cd example && flutter pub get && cd ..
```

---

## Building Release APK

To build a release APK for testing:

```bash
cd example

# Build release APK
flutter build apk --release

# APK location:
# example/build/app/outputs/flutter-apk/app-release.apk
```

**Note**: Release builds require signing. For testing, you can use debug signing or configure signing in `android/app/build.gradle`.

---

## Next Steps

After successful compilation and testing:

1. **Modify Plugin Code**: Edit files in `lib/` or `android/src/main/kotlin/`
2. **Test Changes**: Rebuild and test in example app
3. **Run Tests**: `flutter test` (if tests exist)
4. **Check Linting**: `flutter analyze`

---

## Key Files to Understand

- `lib/background_location.dart`: Dart API that apps use
- `android/src/main/kotlin/.../BackgroundLocationPlugin.kt`: Plugin registration
- `android/src/main/kotlin/.../BackgroundLocationService.kt`: Method channel handler
- `android/src/main/kotlin/.../LocationUpdatesService.kt`: Foreground service implementation
- `example/lib/main.dart`: Example usage of the plugin

---

## Additional Resources

- [Flutter Plugin Development](https://docs.flutter.dev/development/packages-and-plugins/developing-packages)
- [Android Foreground Services](https://developer.android.com/guide/components/foreground-services)
- [Android Location Services](https://developer.android.com/training/location)

