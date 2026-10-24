# Setting up Flutter and Dart on Windows (PowerShell)

This guide walks through installing Flutter, Dart and the related tools on **Windows 10/11 (64-bit)**. Every step is a PowerShell command. At the end you build a **Hello World** Flutter app and a Dart console app to confirm everything works.

> **Dart comes with Flutter.** The Flutter SDK includes the Dart SDK, so installing Flutter gives you both the `flutter` and `dart` commands. You don't need a separate Dart install unless you want Dart without Flutter (see the optional step in Step 3).

---

## Table of Contents

0. [Prerequisites](#0-prerequisites)
1. [Prepare PowerShell](#1-prepare-powershell)
2. [Install Git](#2-install-git)
3. [Install the Flutter SDK (includes Dart)](#3-install-the-flutter-sdk-includes-dart)
4. [Add Flutter to PATH](#4-add-flutter-to-path)
5. [Verify Flutter & Dart](#5-verify-flutter--dart)
6. [Windows desktop toolchain (Visual Studio)](#6-windows-desktop-toolchain-visual-studio)
7. [Web toolchain (Chrome)](#7-web-toolchain-chrome)
8. [Android toolchain (Android Studio, SDK, emulator)](#8-android-toolchain-android-studio-sdk-emulator)
9. [Editor: VS Code + Flutter/Dart extensions](#9-editor-vs-code--flutterdart-extensions)
10. [Run flutter doctor](#10-run-flutter-doctor)
11. [Demo 1: Flutter Hello World app](#11-demo-1-flutter-hello-world-app)
12. [Demo 2: Pure Dart Hello World (console)](#12-demo-2-pure-dart-hello-world-console)
13. [Everyday commands cheat sheet](#13-everyday-commands-cheat-sheet)
14. [Troubleshooting](#14-troubleshooting)
15. [Updating & uninstalling](#15-updating--uninstalling)

---

## 0. Prerequisites

| Requirement | Notes |
|---|---|
| Windows 10 or 11, 64-bit | x64 or ARM64 |
| Free disk space | about 3 GB for Flutter, plus about 10 GB if you add Android Studio and an emulator |
| PowerShell 5.1+ | Built into Windows. PowerShell 7 also works. |
| `winget` | Built into Windows 11 and recent Windows 10 builds (App Installer) |
| Internet access | Needed for the SDK and package downloads |

Check that `winget` is available:

```powershell
winget --version
```

If it's missing, install **App Installer** from the Microsoft Store.

---

## 1. Prepare PowerShell

Open **PowerShell as Administrator**: press Start, type `PowerShell`, then choose **Run as administrator**. Some installers below (Visual Studio, Developer Mode) need admin rights.

Allow locally created scripts to run. Flutter's `.bat` wrappers don't need this, but some tools do:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Turn on **Developer Mode**. Flutter plugins need it to create symlinks on Windows:

```powershell
start ms-settings:developers
# Turn "Developer Mode" ON in the Settings window that opens
```

---

## 2. Install Git

Flutter uses Git to download and update itself.

```powershell
winget install --id Git.Git -e --source winget
```

Close and reopen PowerShell so `git` is on PATH. You can also refresh PATH in the current window:

```powershell
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
git --version
```

Optional, but recommended: allow long file paths. Some Flutter packages have deep folder trees.

```powershell
git config --global core.longpaths true
```

---

## 3. Install the Flutter SDK (includes Dart)

Put Flutter in a folder **without spaces or special characters**, and **not** under `C:\Program Files\` (that folder needs admin rights to write to). This guide uses `C:\src\flutter`.

### Option A (recommended): clone the stable branch with Git

```powershell
New-Item -ItemType Directory -Force -Path C:\src | Out-Null
git clone https://github.com/flutter/flutter.git -b stable C:\src\flutter
```

### Option B: download the zip

1. Download the latest stable Windows zip from
   https://docs.flutter.dev/release/archive?tab=windows
2. Extract it:

```powershell
# Change the file name to match the zip you downloaded
Expand-Archive -Path "$env:USERPROFILE\Downloads\flutter_windows_3.47.8-stable.zip" -DestinationPath C:\src
```

This should create `C:\src\flutter\bin\flutter.bat`.

### Optional: standalone Dart SDK (only if you want Dart without Flutter)

```powershell
# Requires Chocolatey (https://chocolatey.org/install)
choco install dart-sdk -y
```

> If you install standalone Dart, you'll have two Dart SDKs. Whichever appears **first in PATH** wins. For Flutter work, use the one bundled with Flutter.

---

## 4. Add Flutter to PATH

Add `C:\src\flutter\bin` to your **user** PATH so it persists across sessions:

```powershell
$flutterBin = "C:\src\flutter\bin"
$userPath   = [Environment]::GetEnvironmentVariable("Path", "User")

if ($userPath -notlike "*$flutterBin*") {
    [Environment]::SetEnvironmentVariable("Path", "$userPath;$flutterBin", "User")
    Write-Host "Added $flutterBin to user PATH"
} else {
    Write-Host "Flutter already in PATH"
}

# Refresh PATH in the current session
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
```

Confirm the commands resolve to the Flutter folder:

```powershell
(Get-Command flutter).Source   # C:\src\flutter\bin\flutter.bat
(Get-Command dart).Source      # C:\src\flutter\bin\dart.bat
```

---

## 5. Verify Flutter & Dart

The first run downloads the Dart SDK and other build tools into `C:\src\flutter\bin\cache`, so it takes a minute.

```powershell
flutter --version
dart --version
```

Optional: turn off analytics and telemetry.

```powershell
flutter --disable-analytics
dart --disable-analytics
```

---

## 6. Windows desktop toolchain (Visual Studio)

You need this to build **Windows desktop** apps (`flutter run -d windows`). Note that this is **Visual Studio**, not VS Code. It must include the **"Desktop development with C++"** workload.

```powershell
winget install --id Microsoft.VisualStudio.Community -e --source winget `
  --override "--wait --passive --add Microsoft.VisualStudio.Workload.NativeDesktop --includeRecommended"
```

> The install is large (several GB) and takes a while. A newer Visual Studio release also works, as long as `flutter doctor` accepts it.

If Visual Studio is already installed and only the workload is missing:

```powershell
& "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vs_installer.exe" modify `
  --installPath "$env:ProgramFiles\Microsoft Visual Studio\18\Community" `
  --add Microsoft.VisualStudio.Workload.NativeDesktop --includeRecommended --passive
```

---

## 7. Web toolchain (Chrome)

Flutter web runs and debugs in Chrome (Edge also works).

```powershell
winget install --id Google.Chrome -e --source winget
```

If Chrome is installed somewhere non-standard, tell Flutter where it is. Replace the path with your real `chrome.exe`; don't copy a placeholder:

```powershell
[Environment]::SetEnvironmentVariable("CHROME_EXECUTABLE", "C:\Program Files\Google\Chrome\Application\chrome.exe", "User")
```

> Fully close and reopen VS Code afterwards (not just the terminal). Its terminals keep the old value until VS Code restarts.

---

## 8. Android toolchain (Android Studio, SDK, emulator)

Skip this step if you only target Windows or web.

### 8.1 Install Android Studio

```powershell
winget install --id Google.AndroidStudio -e --source winget
```

### 8.2 First-run setup (GUI)

1. Launch **Android Studio**, choose the **Standard** setup, and let it download the Android SDK. The SDK installs to `%LOCALAPPDATA%\Android\Sdk` by default.
2. Go to **More Actions → SDK Manager → SDK Tools** tab and check these:
   - **Android SDK Command-line Tools (latest)** (required by `flutter doctor`)
   - Android SDK Build-Tools
   - Android SDK Platform-Tools
   - Android Emulator
3. Click **Apply**.
4. Optional: go to **Plugins** and install the **Flutter** plugin, which also installs the Dart plugin.

### 8.3 Point Flutter to the SDK and accept licenses

```powershell
$androidSdk = "$env:LOCALAPPDATA\Android\Sdk"
[Environment]::SetEnvironmentVariable("ANDROID_HOME", $androidSdk, "User")
$env:ANDROID_HOME = $androidSdk

flutter config --android-sdk $androidSdk

# Accept all Android SDK licenses (type 'y' at each prompt)
flutter doctor --android-licenses
```

Optional: add `adb` and the emulator to PATH.

```powershell
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$userPath;$androidSdk\platform-tools;$androidSdk\emulator", "User")
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
adb --version
```

### 8.4 Create an Android emulator

**Option A (GUI):** In Android Studio, open **Device Manager → Create Virtual Device**, pick a device (e.g. Pixel), pick a system image, then click **Finish**.

**Option B (PowerShell):** Use `sdkmanager` and `avdmanager`. API level 35 is an example; pick the latest one shown by `sdkmanager --list`.

```powershell
$sdkBin = "$env:LOCALAPPDATA\Android\Sdk\cmdline-tools\latest\bin"

# Download a system image
& "$sdkBin\sdkmanager.bat" "platform-tools" "emulator" "system-images;android-35;google_apis;x86_64"

# Create the virtual device
& "$sdkBin\avdmanager.bat" create avd -n flutter_pixel -k "system-images;android-35;google_apis;x86_64" -d pixel_7
```

List the emulators and start one:

```powershell
flutter emulators
flutter emulators --launch flutter_pixel
```

> To use a **physical Android phone** instead: turn on *Developer options → USB debugging* on the phone, connect it by USB, accept the prompt, then run `flutter devices`.

---

## 9. Editor: VS Code + Flutter/Dart extensions

```powershell
winget install --id Microsoft.VisualStudioCode -e --source winget

# Reopen PowerShell (or refresh PATH) so 'code' is available, then:
code --install-extension Dart-Code.flutter   # also installs Dart-Code.dart-code
code --list-extensions | Select-String -Pattern "dart|flutter"
```

---

## 10. Run flutter doctor

```powershell
flutter doctor -v
```

Expected result, with everything installed:

```
[√] Flutter (Channel stable, ...)
[√] Windows Version (...)
[√] Android toolchain - develop for Android devices (Android SDK version ...)
[√] Chrome - develop for the web
[√] Visual Studio - develop Windows apps (Visual Studio Community 2022 ...)
[√] Android Studio (version ...)
[√] VS Code (version ...)
[√] Connected device (3 available)
[√] Network resources
```

A `[!]` or `[X]` on a platform you don't plan to use (for example Android) is fine. Fix only the ones you need; see [Troubleshooting](#14-troubleshooting).

---

## 11. Demo 1: Flutter Hello World app

### 11.1 Create the project

> Project names must be **lowercase_with_underscores**. Hyphens are not allowed, so use `hello_world`, not `hello-world`.

```powershell
New-Item -ItemType Directory -Force -Path C:\dev | Out-Null
Set-Location C:\dev

flutter create hello_world
Set-Location hello_world
```

To create the project for specific platforms only:

```powershell
# flutter create --platforms=windows,web,android hello_world
```

### 11.2 Replace `lib/main.dart` with a minimal Hello World

```powershell
@'
import 'package:flutter/material.dart';

void main() {
  runApp(const HelloWorldApp());
}

class HelloWorldApp extends StatelessWidget {
  const HelloWorldApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hello World',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int _taps = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Flutter Setup Check')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'Hello, World!',
              style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Text('Button tapped $_taps times', key: const Key('tapCounter')),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _taps++),
        child: const Icon(Icons.add),
      ),
    );
  }
}
'@ | Set-Content -Path .\lib\main.dart -Encoding UTF8
```

### 11.3 Replace the default test so it matches the new app

The generated `test/widget_test.dart` refers to the old `MyApp` counter, so update it as well:

```powershell
@'
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:hello_world/main.dart';

void main() {
  testWidgets('Shows Hello World and increments counter', (tester) async {
    await tester.pumpWidget(const HelloWorldApp());

    expect(find.text('Hello, World!'), findsOneWidget);
    expect(find.text('Button tapped 0 times'), findsOneWidget);

    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(find.text('Button tapped 1 times'), findsOneWidget);
  });
}
'@ | Set-Content -Path .\test\widget_test.dart -Encoding UTF8
```

### 11.4 Get packages, analyze, and test

```powershell
flutter pub get
flutter analyze      # expect: "No issues found!"
flutter test         # expect: "All tests passed!"
```

### 11.5 Run the app

List the available devices:

```powershell
flutter devices
```

Run on a target:

```powershell
flutter run -d windows        # Windows desktop app (needs Step 6)
flutter run -d chrome         # Web, in Chrome (needs Step 7)
flutter run -d emulator-5554  # Android emulator (needs Step 8; use the ID from 'flutter devices')
```

While the app is running, use these keys in the terminal:

| Key | Action |
|---|---|
| `r` | Hot reload (apply code changes instantly) |
| `R` | Hot restart |
| `d` | Detach (leave the app running) |
| `q` | Quit |

**Check hot reload:** With the app running, change `'Hello, World!'` in `lib\main.dart` to `'Hello, Flutter!'`, save the file, and press `r`. The text should update without restarting the app.

### 11.6 Build release artifacts

```powershell
flutter build windows   # -> build\windows\x64\runner\Release\hello_world.exe
flutter build web       # -> build\web\
flutter build apk       # -> build\app\outputs\flutter-apk\app-release.apk
```

Run the built Windows executable:

```powershell
& .\build\windows\x64\runner\Release\hello_world.exe
```

---

## 12. Demo 2: Pure Dart Hello World (console)

This checks that the `dart` CLI works on its own, without Flutter.

```powershell
Set-Location C:\dev
dart create -t console hello_dart
Set-Location hello_dart
```

Replace `bin/hello_dart.dart`:

```powershell
@'
import 'dart:io';

void main(List<String> args) {
  final name = args.isNotEmpty ? args.first : 'World';
  print('Hello, $name!');
  print('Dart ${Platform.version.split(' ').first} on ${Platform.operatingSystem}');
}
'@ | Set-Content -Path .\bin\hello_dart.dart -Encoding UTF8
```

The generated test refers to a sample `calculate()` function. Keep `lib\hello_dart.dart` as it is so `dart test` still passes.

```powershell
dart pub get
dart analyze
dart run                             # Hello, World!  /  Dart 3.x.x on windows
dart run bin\hello_dart.dart Flutter # Hello, Flutter!
dart test
```

Compile to a standalone native `.exe`:

```powershell
dart compile exe bin\hello_dart.dart -o hello_dart.exe
.\hello_dart.exe Flutter
```

---

## 13. Everyday commands cheat sheet

| Command | Purpose |
|---|---|
| `flutter doctor -v` | Check the environment |
| `flutter devices` | List connected devices and targets |
| `flutter emulators --launch <id>` | Start an emulator |
| `flutter create <name>` | Create a new project |
| `flutter pub get` | Install dependencies |
| `flutter pub add <package>` | Add a dependency (e.g. `flutter pub add http`) |
| `flutter pub outdated` / `flutter pub upgrade` | Check or upgrade dependencies |
| `flutter run -d <device>` | Run in debug mode |
| `flutter run --release` | Run in release mode |
| `flutter test` | Run tests |
| `flutter analyze` | Static analysis |
| `dart format .` | Format the code |
| `dart fix --apply` | Apply automated fixes |
| `flutter clean` | Delete build outputs (fixes many odd build errors) |
| `flutter build windows/web/apk/appbundle` | Release builds |
| `flutter upgrade` | Update the Flutter and Dart SDK |
| `flutter channel` | Show or switch channels (`stable`, `beta`, `main`) |

---

## 14. Troubleshooting

| Problem | Fix |
|---|---|
| `flutter : The term 'flutter' is not recognized` | PATH isn't set or refreshed. Redo [Step 4](#4-add-flutter-to-path), then open a **new** PowerShell window. |
| `Building with plugins requires symlink support` | Turn on Developer Mode: `start ms-settings:developers` |
| `cmdline-tools component is missing` | Android Studio → SDK Manager → SDK Tools → check **Android SDK Command-line Tools (latest)** |
| `Android license status unknown` | `flutter doctor --android-licenses` |
| `Unable to locate Android SDK` | `flutter config --android-sdk "$env:LOCALAPPDATA\Android\Sdk"` |
| Visual Studio is missing the C++ workload | Rerun the `vs_installer.exe modify ... NativeDesktop` command in [Step 6](#6-windows-desktop-toolchain-visual-studio) |
| `Cannot find Chrome executable` | Set `CHROME_EXECUTABLE` ([Step 7](#7-web-toolchain-chrome)) |
| Emulator is very slow or won't start | Turn on virtualization (VT-x/AMD-V) in BIOS, and turn on Windows **Hypervisor Platform**: `Enable-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform` (as admin, then reboot) |
| Filename too long errors | `git config --global core.longpaths true`, and turn on Win32 long paths (admin): `New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name LongPathsEnabled -Value 1 -PropertyType DWORD -Force` |
| Downloads fail behind a corporate proxy | `$env:HTTPS_PROXY="http://proxy:port"; $env:HTTP_PROXY="http://proxy:port"; $env:NO_PROXY="localhost,127.0.0.1"` |
| Strange build or Gradle errors | `flutter clean; flutter pub get`, then try again |
| Antivirus slows builds a lot | Exclude `C:\src\flutter` and your project folder from real-time scanning |

---

## 15. Updating & uninstalling

### Update

```powershell
flutter upgrade
flutter doctor
```

### Uninstall Flutter and Dart

```powershell
# Remove the SDK and caches
Remove-Item -Recurse -Force C:\src\flutter
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Pub\Cache" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:APPDATA\.dart", "$env:APPDATA\.dart-tool", "$env:APPDATA\.flutter-devtools" -ErrorAction SilentlyContinue
Remove-Item -Force "$env:USERPROFILE\.flutter", "$env:USERPROFILE\.flutter_tool_state" -ErrorAction SilentlyContinue

# Remove the Flutter folder from the user PATH
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
$newPath  = ($userPath -split ';' | Where-Object { $_ -and $_ -notlike "*\flutter\bin*" }) -join ';'
[Environment]::SetEnvironmentVariable("Path", $newPath, "User")
```

Remove the other tools if you no longer need them:

```powershell
winget uninstall --id Google.AndroidStudio
winget uninstall --id Microsoft.VisualStudio.2022.Community
winget uninstall --id Microsoft.VisualStudioCode
```

---

### Quick setup summary

```powershell
winget install --id Git.Git -e
git clone https://github.com/flutter/flutter.git -b stable C:\src\flutter
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";C:\src\flutter\bin", "User")
# reopen PowerShell
flutter doctor -v
flutter create hello_world; cd hello_world
flutter run -d windows   # or: -d chrome
```

References: https://docs.flutter.dev/get-started/install/windows · https://dart.dev/get-dart



git config --global user.name "danasalah2505"
git config --global user.email "danasalah2505@gmail.com"
