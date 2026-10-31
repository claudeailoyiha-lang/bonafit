name: Build FitEmpire APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Prepare and Build FitEmpire APK
        run: |
          if ls FitEmpire-mobile-main*.zip 1> /dev/null 2>&1; then
            for z in FitEmpire-mobile-main*.zip; do unzip -oq "$z" -d extracted; done
          else
            for z in *.zip; do unzip -oq "$z" -d extracted; done
          fi
          flutter create --org uz.fitempire --project-name fitempire --platforms android fitempire_build
          MAIN_DART=$(find extracted -name "main.dart" | head -n 1)
          RES_DIR=$(dirname "$(find extracted -type d -name "mipmap-hdpi" | head -n 1)")
          cp "$MAIN_DART" fitempire_build/lib/main.dart
          rm -rf fitempire_build/android/app/src/main/res/mipmap-anydpi*
          cp -r "$RES_DIR"/mipmap-* fitempire_build/android/app/src/main/res/
          sed -i 's/android:label="fitempire"/android:label="FitEmpire"/g' fitempire_build/android/app/src/main/AndroidManifest.xml
          cd fitempire_build
          flutter build apk --release
          cp build/app/outputs/flutter-apk/app-release.apk ../FitEmpire.apk

      - name: Upload FitEmpire APK
        uses: actions/upload-artifact@v4
        with:
          name: FitEmpire-Tayyor-APK
          path: FitEmpire.apk
