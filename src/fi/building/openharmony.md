# Kääntäminen OpenHarmonylle

<div class="warning _note">
OpenHarmony-tuki on parhaillaan kehitteillä, ja nämä ohjeet voivat muuttua ajoittain ja olla myös puutteellisia.
</div>

## OpenHarmony-työkalujen hankkiminen

OpenHarmonylle kääntäminen vaatii seuraavat:

1. OpenHarmony SDK. Tämä riittää Servon kääntämiseen jaetulla kirjastolla OpenHarmonylle.
2. `hvigor`-käännöstyökalun sovelluksen kääntämiseen sovelluspaketiksi ja allekirjoittamiseen.

### OpenHarmony SDK:n asennus

OpenHarmony SDK vaaditaan sovellusten kääntämiseen OpenHarmonylle.
Servon tukeman SDK:n vähimmäisversio on v6.0.0 (API-20).

#### Lataus DevEco Studion kautta

[DevEco Studio] on IDE HarmonyOS NEXT- ja OpenHarmony-sovellusten kehittämiseen.
Se tukee Windowsia ja macOS:ää.
Voit hallita asennettuja OpenHarmony SDK:ita valitsemalla File->Settings ja "OpenHarmony SDK".
Kun olet asettanut sopivan asennuspolun, voit valita asennettavat komponentit kullekin saatavilla olevalle API-versiolle.
DevEco Studio lataa ja asentaa komponentit automaattisesti.


#### OpenHarmony SDK:n manuaalinen asennus (esim. Linuxilla)

<div class="warning _note">
    Ennen kuin lataat OH SDK:n giteestä kuten tässä kuvataan, huomaa että tarvitset myös hvigorin sovellusten kääntämiseen.
    hvigor suositellaan ladattavaksi HarmonyOS NEXT -komentorivityökalupaketin kautta, joka sisältää myös kopion OpenHarmony SDK:sta.
</div>

1. Mene [OpenHarmony release notes] -sivulle ja valitse versio, jolle haluat kääntää.
2. Vieritä osioon "Acquiring Source Code from Mirrors" ja klikkaa latauslinkkiä "Public SDK package for the standard system" -versiolle, joka vastaa isäntäjärjestelmääsi.
3. Pura arkisto sopivaan sijaintiin.
4. Siirry SDK-kansioon komennolla `cd <sdk_folder>/<your_operating_system>`.
5. Luo alikansio samalla nimellä kuin API-versio (esim. 14 SDK v5.0.2:lle) ja siirry siihen.
6. Pura yksittäisten komponenttien zip-tiedostot edellisessä vaiheessa luotuun kansioon. Käytä mieluiten `unzip`-komentoa komentorivillä, tai varmista manuaalisesti että puretut paketit ovat nimeltään esim. `native` eivätkä `native-linux-x64-5.x.y.z`.

Seuraava pätkä toimii viitteenä vaiheille 4–6:
```commandline
cd ~/ohos-sdk/linux
for COMPONENT in "native toolchains ets js previewer" do
    echo "Extracting component ${COMPONENT}"
    unzip ${COMPONENT}-*.zip
    API_VERSION=$(cat ${COMPONENT}/oh-uni-package.json | jq -r '.apiVersion')
    mkdir -p ${API_VERSION}
    mv ${COMPONENT} "${API_VERSION}/"
done
```
Windowsilla on suositeltavaa käyttää 7zip:iä arkistojen purkamiseen, koska Windows Explorerin purkutyökalu on erittäin hidas.

[DevEco Studio]: https://developer.huawei.com/consumer/cn/deveco-studio
[OpenHarmony release notes]: https://gitee.com/openharmony/docs/tree/master/en/release-notes/
[HarmonyOS NEXT commandline tools]: https://developer.huawei.com/consumer/cn/download/

#### HarmonyOS NEXT -komentorivityökalujen manuaalinen asennus

[HarmonyOS NEXT commandline tools] sisältää OpenHarmony SDK:n ja seuraavat lisätyökalut:

- `codelinter` (linter)
- `hstack` (crash dump stack analysis tool)
- `hvigor` / `hvigorw` (build tool)
- `ohpm` (package manager)

Tällä hetkellä komentorivityökalupaketti ei ole julkisesti saatavilla ja vaatii kiinalaisen Huawei-tilin lataamiseen.

#### hvigorin manuaalinen asennus ilman komentorivityökaluja

<div class="warning _note">
Tätä osiota ei ole täysin testattu, ja se voi muuttua käyttäjäpalautteen perusteella.
Komentorivityökalupaketin asentaminen on suositeltavaa. Jos päätät asentaa manuaalisesti, sinun täytyy
huolehtia hvigor-version asentamisesta, joka vastaa projektisi vaatimuksia.
</div>

`hvigor` (ei wrapper `hvigorw`) on saatavilla myös `npm`:n kautta.
1. Asenna sama nodejs-versio kuin komentorivityökalut toimittavat.
   HarmonyOS NEXT:lle toimitetaan Node 18. Varmista, että `node`-binääri on PATH:ssa.
2. Asenna Java suositellulla asennustavalla käyttöjärjestelmällesi.
   Käännösvaiheet toimivat tunnetusti OpenJDK v17, v21 ja v23:lla.
   macOS:lla, jos asennat Homebrew'n [OpenJDK formula] -kaavan, seuraava lisäkomento saattaa olla tarpeen asennuksen jälkeen:
   ```
   # For the system Java wrappers to find this JDK, symlink it with
   sudo ln -sfn $HOMEBREW_PREFIX/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
   ```
2. Muokkaa `.npmrc`-tiedostoasi sisältämään seuraava rivi:

    ```
    @ohos:registry=https://repo.harmonyos.com/npm/
    ```

3. Asenna hvigor ja hvigor-ohos-plugin. Tämä luo `node_modules`-hakemiston nykyiseen hakemistoon.
   ```commandline
   npm install @ohos/hvigor
   npm install @ohos/hvigor-ohos-plugin
   ```
4. Nyt sinun pitäisi pystyä ajamaan `hvigor.js` OpenHarmony-projektissasi kääntääksesi hap-paketin:
   ```
   /path/to/node_modules/@ohos/hvigor/bin/hvigor.js assembleHap
   ```
[OpenJDK formula]: https://formulae.brew.sh/formula/openjdk#default

### hdc:n konfigurointi Linuxilla

`hdc` on OpenHarmony-laitteiden vastine `adb`:lle.
Löydät sen SDK:n `toolchains`-hakemistosta.
Käytännön syistä saatat haluta lisätä `toolchains`in `PATH`:iin.
Muun muassa `hdc`:llä voi avata shellin tai siirtää tiedostoja laitteen ja isäntäjärjestelmän välillä.
`hdc` tarvitsee yhteyden fyysiseen laitteeseen `usb`:n kautta, mikä vaatii käyttäjältä oikeudet laitteen käyttöön.

On suositeltavaa lisätä `udev`-sääntö, jotta hdc pääsee laitteeseen ilman root-oikeuksia.
[Tämä stackoverflow-vastaus](https://stackoverflow.com/a/53887437) pätee myös `hdc`:hen.
Aja `lsusb` ja tarkista laitteesi vendor id, luo sitten vastaava `udev`-sääntö.
Huomaa, että käyttäjäsi tulee olla `GROUP="xxx"`:llä määritetyn ryhmän jäsen.
Jakelustasi riippuen saatat haluta käyttää eri ryhmää.

Tarkistaaksesi toimiiko `hdc`, aja `hdc list targets` — sen pitäisi näyttää laitteesi sarjanumero.
Jos se ei toimi, kokeile uudelleenkäynnistystä.

Huomaa, että laitteesi täytyy olla "Developer mode" -tilassa USB-debuggaus päällä.
Prosessi on täsmälleen sama kuin Androidilla:
1. Napauta build-numeroa useita kertoja ottaaksesi kehittäjätilan käyttöön.
2. Siirry sitten kehittäjäasetuksiin ja ota USB-debuggaus käyttöön.
3. Kun yhdistät laitteen ensimmäistä kertaa, vahvista ponnahdusikkuna, jossa kysytään luotatko yhdistettävään tietokoneeseen.

## Allekirjoitusasetukset

Useimmat laitteet vaativat, että HAP on digitaalisesti allekirjoitettu kehittäjän toimesta asennusta varten.
`hvigor`-työkalulla tämä onnistuu asettamalla staattinen `signingConfigs`-objekti `build-profile.json5`-tiedostoon tai luomalla dynaamisesti `signingConfigs`-taulukko sovelluskontekstiobjektille `hvigorfile.ts`-skriptissä.

`signingConfigs`-ominaisuus on taulukko objekteja seuraavalla rakenteella:

```json
[
    {
        "name": "default",
        "type": "<OpenHarmony or HarmonyOS>",
        "material": {
            "certpath": "/path/to/app-signing-certificate.cer",
            "storePassword": "<encrypted password>",
            "keyAlias": "debugKey",
            "keyPassword": "<encrypted password>",
            "profile": "/path/to/signed-profile-certificate.p7b",
            "signAlg": "SHA256withECDSA",
            "storeFile": "/path/to/java-keystore-file.p12"
        }
    }
]
```

Tässä `<encrypted password>` on salasanan plaintext-esityksen heksadesimaalimerkkijono salauksen jälkeen.
Avain ja suola salasanojen salaamiseen luodaan DevEco Studio IDE:llä, ja ne tallennetaan levylle sertifikaattitiedostojen ja keystoren rinnalle, yleensä hakemistoon `<USER HOME>/.ohos`.

Luodaksesi salauksessa tarvittavat tiedot, vaaditut sovellus- ja profiilisertifikaattitiedostot ja itse keystoren, voit kloonata [esimerkki-ArkTS-sovelluksen](https://github.com/jschwe/ServoDemo) ja avata sen DevEco Studio IDE:llä.
Huomaa, että koska allekirjoitustiedot sidotaan bundle-nimeen, kaikki ArkTS-sovellukset eivät toimi, ja siksi on **erittäin** suositeltavaa käyttää yllä mainittua esimerkki-ArkTS-sovellusta.

1. Avaa Project Structure -valintaikkuna valikosta `File > Project Structure`.
2. Välilehdellä 'Signing Config' ota käyttöön 'Automatically generate signature' -valintaruutu.

**HUOM: Yllä automaattisesti luotu allekirjoitus on tarkoitettu vain kehitykseen ja testaukseen. Tuotantokäännöksiin ja jakeluun sovelluskaupan kautta tarvitaan asianmukainen konfiguraatio sovelluskaupan tarjoajalta.**

>Linux-käyttäjille DevEco Studio on saatavilla vain Windowsilla ja macOS:lla. Jatkaaksesi **tarvitset toisen Windows- / macOS-koneen, jossa DevEco Studio IDE on asennettuna** allekirjoitusavainten luomiseen. Jos kehität OpenHarmony-levyjä (kuten HopeRun-kehityslevy), voit nimetä `SigningConfigs`:in `default`. Muuten aseta se arvoon `hos`, jos kehität Servoa HarmonyOS-laitteille (kuten Huawei Mate -puhelinsarja).
>
> Kun avaimet on luotu, sinun täytyy siirtää koko hakemisto, joka tallentaa avaimet (yleensä `<USER HOME>/.ohos/`), DevEco Studion luomalta Windows- / macOS-koneelta. 
>
> Lisäksi sinun täytyy kopioida `SigningConfigs` DevEco Studion luomasta `build-profile.json5`:stä Windows- / macOS-koneelta `.json`-tiedostoon Linux-koneellasi. Tämä toimii "signing material" -lähteenä, johon `mach` voi myöhemmin viitata.

Kun luotu, sinun täytyy osoittaa `mach` yllä olevaan "signing material" -konfiguraatioon `SERVO_OHOS_SIGNING_CONFIG`-ympäristömuuttujalla.
Muuttujan arvon täytyy olla polku kelvolliseen `.json`-tiedostoon, jolla on sama rakenne kuin yllä olevalla `signingConfigs`-ominaisuudella, mutta jossa `certPath`, `storeFile` ja `profile` on annettu *polkuina suhteessa json-tiedostoon* absoluuttisten polkujen sijaan.


## servoshellin kääntäminen

Ennen Servon kääntämistä sinun täytyy asettaa joitakin ympäristömuuttujia.
[direnv](https://direnv.net) on kätevä työkalu, joka voi asettaa nämä muuttujat automaattisesti `.envrc`-tiedoston perusteella, mutta voit käyttää mitä tahansa muuta tapaa asettaaksesi vaaditut ympäristömuuttujat.

`.envrc`:
```commandline
    export OHOS_SDK_NATIVE=/path/to/openharmony-sdk/platform/api-version/native

    # Required if the HAP must be signed. See the signing configuration section above.
    export SERVO_OHOS_SIGNING_CONFIG=/path/to/signing-configs.json

    # Required only when building for HarmonyOS:
    export DEVECO_SDK_HOME=/path/to/command-line-tools/sdk # OR /path/to/DevEcoStudio/sdk OR on MacOS /Applications/DevEco-Studio.app/Contents/sdk

    # Required only when building for OpenHarmony:
    # Note: The openharmony sdk is under ${DEVECO_SDK_HOME}/default/openharmony
    # Presumably you would need to replicate this directory structure
    export OHOS_BASE_SDK_HOME=/path/to/openharmony-sdk/platform

    # If you have the command-line tools installed:
    export PATH="${PATH}:/path/to/command-line-tools/bin/"
    export NODE_HOME=/path/to/command-line-tools/tool/node

    # Alternatively, if you do NOT have the command-line tools installed:
    export HVIGOR_PATH=/path/to/parent/directory/containing/node_modules  # Not required if `hvigorw` is in $PATH
```

Jos käytät `direnv`:iä ja `.envrc`-tiedostoa, muista ajaa `direnv allow .` `.envrc`-tiedoston muokkaamisen jälkeen.
Muuten ympäristömuuttujia ei ladata.

Seuraavalla komennolla voit kääntää servoshell-sovelluksen 64-bittiselle ARM-laitteelle tai emulaattorille:

```commandline
./mach build --ohos --release [--flavor=harmonyos]
```

Komennoissa `mach build`, `mach install` ja `mach package` `--ohos` on alias `--target aarch64-unknown-linux-ohos`:lle.
Kääntääksesi emulaattorille, joka ajaa x86-64-isännällä, käytä `--target x86_64-unknown-linux-ohos`.
Oletus `ohos` -käännös / pakkaus / asennus kohdistaa OpenHarmonyyn.
Jos haluat kääntää HarmonyOS:lle, voit lisätä `--flavor=harmonyos`.
Tarkista [Allekirjoitusasetukset](#allekirjoitusasetukset) ja lisää konfiguraatio, jossa `"name": "hos"` ja `"type": "HarmonyOS""` ja vastaavat allekirjoitussertifikaatit.


## Asentaminen ja ajaminen laitteella

Seuraavalla komennolla voit asentaa aiemmin käännetyn servoshell-sovelluksen 64-bittiselle ARM-laitteelle tai emulaattorille:

```commandline
./mach install --ohos --release [--flavor=harmonyos]
```

## Lisälukemista

[OpenHarmony Glossary](https://gitee.com/openharmony/docs/tree/master/en/glossary.md)

## Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia.
