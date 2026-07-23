# Dateimanager

Android-Dateimanager als Fallstudienprojekt im Bereich hybride App-Entwicklung.  
Die App wurde mit Ionic, Vue, TypeScript und Capacitor umgesetzt.

## Funktionen

- Dateien und Ordner des aktuellen Verzeichnisses anzeigen
- Dateien und Ordner durch unterschiedliche Symbole kennzeichnen
- Dateien über den Android-Dateiauswahldialog importieren
- importierte Dateien in das interne App-Verzeichnis kopieren
- neue Ordner mit verpflichtender Namenseingabe erstellen
- Ordner öffnen und in Unterordner navigieren
- eine Ordnerebene zurücknavigieren
- Dateien mit einer geeigneten Android-App öffnen
- Dateien und Ordner nach Bestätigung löschen
- gefüllte Ordner einschließlich ihres Inhalts löschen

Benutzerdateien werden innerhalb von `Directory.Data/dateimanager-files` gespeichert. Dadurch werden technische Dateien der App nicht in der Dateiliste angezeigt.

## Technische Voraussetzungen

- Windows
- Node.js 24
- npm 11
- JDK 21
- Android SDK
- Android Platform 36
- Android Build Tools 36.1.0
- Android Platform Tools
- Android Emulator
- System-Image `system-images;android-36;google_apis;x86_64`
- ausreichend freier Speicherplatz
- Zugriff auf die private npm-Abhängigkeit von Capawesome

Das Projekt verwendet eine private Capawesome-Abhängigkeit. Die erforderliche npm-Authentifizierung wird dem Teammitglied separat bereitgestellt und darf ausschließlich in der benutzerspezifischen npm-Konfiguration gespeichert werden. Lizenzschlüssel und npm-Token dürfen niemals in das Repository, die README oder eine Projektdatei geschrieben werden.

## Ersteinrichtung auf einem neuen Windows-Rechner

### 1. Voraussetzungen installieren

Node.js 24 einschließlich npm, Eclipse Temurin JDK 21 und die Android SDK Command-Line Tools installieren. Die Command-Line Tools müssen in folgender Struktur bereitgestellt werden:

```text
%LOCALAPPDATA%\Android\Sdk\cmdline-tools\latest
```

### 2. Repository-ZIP entpacken

Das ZIP vollständig entpacken und PowerShell im Stammverzeichnis des entpackten Repositorys öffnen.

### 3. Private npm-Konfiguration einrichten

Die private Capawesome-Konfiguration muss vor `npm.cmd ci` durch ein Teammitglied bereitgestellt werden. Sie wird in der benutzerspezifischen npm-Konfiguration gespeichert und darf weder in den Repository-Ordner kopiert noch committed werden.

### 4. Temporäre Umgebung setzen

```powershell
$env:JAVA_HOME = '<PFAD-ZUM-INSTALLIERTEN-JDK-21>'
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"

$env:Path = "$env:JAVA_HOME\bin;" +
            "$env:ANDROID_HOME\platform-tools;" +
            "$env:ANDROID_HOME\emulator;" +
            "$env:ANDROID_HOME\cmdline-tools\latest\bin;" +
            $env:Path
```

`<PFAD-ZUM-INSTALLIERTEN-JDK-21>` muss durch den tatsächlichen Installationspfad des JDK 21 ersetzt werden.

```powershell
java -version
npm -version
sdkmanager.bat --version
adb version
```

Java muss Version 21 anzeigen.

### 5. Android-SDK-Pakete installieren

```powershell
sdkmanager.bat --sdk_root="$env:ANDROID_HOME" `
  "platform-tools" `
  "platforms;android-36" `
  "build-tools;36.1.0" `
  "emulator" `
  "system-images;android-36;google_apis;x86_64"

sdkmanager.bat --sdk_root="$env:ANDROID_HOME" --licenses
```

### 6. Emulator erstellen

```powershell
avdmanager.bat create avd `
  -n Dateimanager_API_36 `
  -k "system-images;android-36;google_apis;x86_64"
```

Bei der Frage nach einer eigenen Hardwarekonfiguration kann die Standardauswahl verwendet werden. Falls das AVD `Dateimanager_API_36` bereits existiert, darf dieser Schritt übersprungen werden.

### 7. Projektabhängigkeiten installieren

Im Repository-Stamm:

```powershell
npm.cmd ci
```

### 8. Projekt bauen und Android synchronisieren

```powershell
npm.cmd run build
npx.cmd cap sync android
```

### 9. Debug-APK bauen

```powershell
cd android
.\gradlew.bat assembleDebug
cd ..
```

### 10. Emulator starten

```powershell
Start-Process `
  -FilePath "$env:ANDROID_HOME\emulator\emulator.exe" `
  -ArgumentList '-avd', 'Dateimanager_API_36', '-no-snapshot-load'

adb wait-for-device
adb shell getprop sys.boot_completed
```

Der Wert von `sys.boot_completed` muss `1` sein.

### 11. APK installieren und App starten

```powershell
adb install -r "android\app\build\outputs\apk\debug\app-debug.apk"
adb shell am start -n io.ionic.starter/.MainActivity
```

## Start auf einem bereits vorbereiteten Rechner

Dieser Ablauf gilt, wenn Node.js und npm, JDK 21, das Android SDK, die benötigten SDK-Pakete, die private npm-Konfiguration und das AVD `Dateimanager_API_36` bereits vorhanden sind.

### 1. ZIP entpacken und PowerShell öffnen

Das ZIP vollständig entpacken und PowerShell im Stammverzeichnis des entpackten Repositorys öffnen.

### 2. Temporäre Umgebung setzen

```powershell
$env:JAVA_HOME = '<PFAD-ZUM-INSTALLIERTEN-JDK-21>'
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"

$env:Path = "$env:JAVA_HOME\bin;" +
            "$env:ANDROID_HOME\platform-tools;" +
            "$env:ANDROID_HOME\emulator;" +
            "$env:ANDROID_HOME\cmdline-tools\latest\bin;" +
            $env:Path
```

### 3. Abhängigkeiten installieren und Projekt bauen

```powershell
npm.cmd ci
npm.cmd run build
npx.cmd cap sync android

cd android
.\gradlew.bat assembleDebug
cd ..
```

### 4. Emulator starten

```powershell
Start-Process `
  -FilePath "$env:ANDROID_HOME\emulator\emulator.exe" `
  -ArgumentList '-avd', 'Dateimanager_API_36', '-no-snapshot-load'

adb wait-for-device
```

### 5. APK installieren und App starten

```powershell
adb install -r "android\app\build\outputs\apk\debug\app-debug.apk"
adb shell am start -n io.ionic.starter/.MainActivity
```

## APK-Pfad

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

Android-Buildausgaben sind normalerweise nicht Bestandteil des GitHub-ZIPs. Die APK muss deshalb entweder selbst gebaut oder separat vom Team bereitgestellt werden.

## Verwendete Technologien

- Ionic
- Vue
- TypeScript
- Capacitor
- Android
- Capacitor Filesystem
- Capawesome File Picker
- Capawesome File Opener
