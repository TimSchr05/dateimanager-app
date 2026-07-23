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

## Voraussetzungen

- Node.js 24
- npm 11
- JDK 21
- Android SDK mit Android-Plattform 36
- Android-Emulator oder Android-Gerät

## Projekt installieren

```powershell
npm install
```

## Web-Anwendung bauen und Android synchronisieren

```powershell
npm run build
npx cap sync android
```

## Android-Debug-APK erstellen

```powershell
$env:JAVA_HOME = 'C:\Program Files\Eclipse Adoptium\jdk-21.0.11.10-hotspot'
cd android
.\gradlew.bat assembleDebug
cd ..
```

Die erzeugte APK befindet sich unter:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

## App im vorhandenen Emulator starten

Das verwendete virtuelle Gerät heißt `Dateimanager_API_36`.

```powershell
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"

& "$env:ANDROID_HOME\emulator\emulator.exe" `
  -avd Dateimanager_API_36 `
  -no-snapshot-load
```

Nach dem vollständigen Start des Emulators:

```powershell
& "$env:ANDROID_HOME\platform-tools\adb.exe" install -r `
  "android/app/build/outputs/apk/debug/app-debug.apk"

& "$env:ANDROID_HOME\platform-tools\adb.exe" shell am start `
  -n io.ionic.starter/.MainActivity
```

## Verwendete Technologien

- Ionic
- Vue
- TypeScript
- Capacitor
- Android
- Capacitor Filesystem
- Capawesome File Picker
- Capawesome File Opener
