# EZQ Kiosk app - build steps (Windows)

One-time installs: Node.js LTS (nodejs.org) and Android Studio (developer.android.com/studio).

1. Open Command Prompt in this folder (kiosk-app).
2. `npm install`
3. `npx cap add android`
4. `npx cap sync android`
5. `npx cap open android` - Android Studio opens. Wait for Gradle to finish.
6. Open `android/app/src/main/AndroidManifest.xml` and make sure these lines exist inside <manifest>:
   <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
   <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
   <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
   <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
   <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
7. Build > Build APK(s). Copy `app-debug.apk` to the tablet and install it
   (allow "install unknown apps"), or plug the tablet in and press Run.
8. On the tablet: pair the printer in Android Bluetooth settings first.
   Open the app, sign in as the kiosk account, tap the faint "Printer" button
   (bottom-left), Search, pick the printer, Test print.

The app loads the live website, so changes to customer.html only need `firebase deploy`.
Rebuild the APK only if you change this folder (name, icon, plugins).
Keep screen on / pin the app: Android Settings > Security > App pinning (screen pinning).
