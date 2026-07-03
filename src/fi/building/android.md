# Kääntäminen Androidille

- Varmista, että seuraavat ympäristömuuttujat on asetettu:
  - `ANDROID_SDK_ROOT`
  - `ANDROID_NDK_ROOT`: `$ANDROID_SDK_ROOT/ndk/28.2.13676358/`
 `ANDROID_SDK_ROOT` voi olla mikä tahansa hakemisto (kuten `~/android-sdk`).
  Kaikki Android-käännösriippuvuudet asennetaan sinne.
- Asenna [Android command-line tools](https://developer.android.com/studio#command-tools) -työkalujen uusin versio hakemistoon `$ANDROID_SDK_ROOT/cmdline-tools/latest`.
- Asenna tarvittavat komponentit seuraavalla komennolla:
  ```shell
  sudo $ANDROID_SDK_ROOT/cmdline-tools/latest/bin/sdkmanager --install \
   "build-tools;36.0.0" \
   "emulator" \
   "ndk;28.2.13676358" \
   "platform-tools" \
   "platforms;android-37" \
   "system-images;android-37;google_apis;arm64-v8a"
  ```
- Aja `./mach build --android`

**Huom**: Tämä asentaa riippuvuudet ja kääntää Servon `aarch64-linux-android` -alustalle.
Kääntääksesi Servon muille Android-kohteille, asenna sopivat järjestelmäkuvat `sdkmanager`:lla ja anna `mach`:lle `--target` Rust-yhteensopivalla kohteella `--android`:n sijaan.

**Huom**: Jos et käytä Android Studioa macOS:lla, sinun täytyy asentaa JDK.
Käytä `brew install opendjdk@21` asentaaksesi toimivan version; uudemmat versiot aiheuttavat `java.lang.IllegalArgumentException: 25` Servon käännöksen gradle-vaiheessa.


**Huom**: Jos käytät Nixiä, sinun ei tarvitse asentaa työkaluja tai asettaa `ANDROID_SDK_ROOT`- ja `ANDROID_NDK_ROOT` -ympäristömuuttujia manuaalisesti.
Ota Android-käännöstuki käyttöön ajamalla:

```
export SERVO_ANDROID_BUILD=1
```

shell-istunnossa ennen `./mach`-komentoja

## Kääntäminen Android Studiolla

Servon kääntäminen komentoriviltä on suositeltavaa Androidille, mutta voit kääntää sen myös Android Studiolla, jos haluat käyttää Android-IDE:tä.

- Asenna Android Studio lataamalla sopiva versio [viralliselta sivustolta](https://developer.android.com/) ja seuraamalla [asennusohjeita](https://developer.android.com/studio/install).
- Asentaaksesi lisätyökaluja, käynnistä Android Studio, mene *Settings*-valikkoon ja kirjoita hakupalkkiin `sdk`.
- Valitse SDK:
  - Klikkaa *Android SDK`*
  - Siirry kohtaan *SDK Tools*
  - Valitse *Android SDK Command-line tools (latest)*: ![image](../../images/android-additional-downloads.png)
- Valitse NDK. Huomaa, että Servo **vaatii NDK:n version 28**.
  - Osiossa *SDK Tools* valitse *NDK (side by side)*
  - Klikkaa *show package details*: ![image](../../images/android-sdk-details-1.png)
  - Valitse NDK 28:n uusin versio: ![image](../../images/android-sdk-details-2.png)
- Klikkaa *Ok* asentaaksesi sekä NDK:n että SDK:n.
- Etsi SDK:n polku kohdasta *Languages & Frameworks* sitten *Android SDK Location*: ![image](../../images/android-sdk-path.png).
  Varmista sitten, että seuraavat ympäristömuuttujat on asetettu:
  - `ANDROID_SDK_ROOT`: Yllä löytynyt polku.
  - `ANDROID_NDK_ROOT`: `$ANDROID_SDK_ROOT/ndk/28.2.13676358/`
- Aja `./mach build --android`

## Ajaminen emulaattorissa

1. Luo uusi AVD-kuva Servon ajamiseen:
    ```
    $ANDROID_SDK_ROOT/cmdline-tools/latest/bin/avdmanager create avd \
        --name "Servo" \
        --device "pixel" \
        --package "system-images;android-33;google_apis;arm64-v8a" \
        --tag "google_apis" \
        --abi "arm64-v8a"
    ```
2. Ota laitteiston näppäimistö käyttöön.
   Avaa `~/.android/avd/Servo.avd/config.ini` ja muuta `hw.keyboard = no` muotoon `hw.keyboard = yes`.
3. Käynnistä emulaattori
   ```
   $ANDROID_SDK_ROOT/emulator/emulator -avd Servo -netdelay none -no-snapshot
   ```
4. Asenna Servo emulaattoriin:
   ```
    ./mach install --android
   ```
5. Käynnistä Servo napauttamalla Servo-kuvaketta käynnistysohjelmassa.

## Asentaminen fyysiselle laitteelle

1. [Valmistele laitteesi kehitystä varten](https://developer.android.com/studio/run/device).
2. Käännä Servo kuten yllä, varmistaen että käännät sopivalle kohteelle laitteellesi.
3. Asenna Servo laitteellesi ajamalla:
   ```
   ./mach install --android
   ```
4. Käynnistä Servo napauttamalla Servo-kuvaketta käynnistysohjelmassa tai aja:
   ```
   ./mach run --android https://www.servo.org/
   ```

Voit pakottaa Servon pysähtymään ajamalla:
```
adb shell am force-stop org.servo.servoshell/org.servo.servoshell.MainActivity
```

Jos yllä oleva ei toimi, kokeile tätä:
```
adb shell am force-stop org.servo.servoshell
```

Voit poistaa Servon asennuksen ajamalla:
```
adb uninstall org.servo.servoshell
```

## Vianmääritys 

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia.
