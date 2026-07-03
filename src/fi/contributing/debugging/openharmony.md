# Debuggaus OpenHarmonylla

On suositeltavaa lukea ensin yleiset ohjeet [Debuggaus-osiosta](index.md).
Tämä osio käsittelee vain OpenHarmonyyn liittyviä eroja.

## Servoshellin käynnistäminen komentoriviltä

Debuggauksessa on usein hyödyllistä käynnistää servo komentoriviltä sovelluskuvakkeen sijaan, koska voimme välittää parametreja komentoriviltä.

```shell
# Run this command to see which parameters can be passed to an ohos app
hdc shell aa start --help
# Start servo from the commandline
hdc shell aa start -a EntryAbility -b org.servo.servo
# --ps=<arg> <value> can be used to pass arguments with values to servoshell.
# The space between arg and value is mandatory when using `--ps`, meaning `--ps=--log-filter warn` is
# correct while `--ps=--log-filter=warn` is not.
# For pure flags without a value use `--psn <flag>`
hdc shell aa start -a EntryAbility -b org.servo.servo --ps=--log-filter "warn"
# Use `-U <url>` to let servo load a custom URL.
hdc shell aa start -a EntryAbility -b org.servo.servo -U https://servo.org
```

## Lokitus

OpenHarmony-laitteilla lokiviestit voidaan tallentaa ja hakea `hilog`-palvelun kautta:

```shell
# See the hilog help for a full list of arguments supported by hilog
hdc shell hilog --help
# View all logs (very verbose, includes all other apps)
hdc shell hilog
# All servo and Spidermonkey related logs
hdc shell hilog --domain=0xE0C3,0xE0C4
# Only Rust code from servo
hdc shell hilog --domain=0xE0C3
# Only spidermonkey C++ code
hdc shell hilog --domain=0xE0C4
# Filter by domain and log level
hdc shell hilog --domain=0xE0C3 --level=ERROR
```

### Lokitaso

Riippuu useista ehdoista, näkyvätkö lokiviestit:

1. `log`-kirjaston käännösaikainen maksimilokitaso. Katso [log compile-time filters].
  Huom: `log`-kirjaston `release_max_level_<level>`-ominaisuudet tarkistavat, onko `debug_assertions = false` asetettu
  määrittääkseen, sovelletaanko release-suodattimia.
2. Ajonaikainen `log`-kirjaston globaali suodatin moduuleittain, joka on asetettu servoshellissä. Koska ympäristömuuttujat eivät ole
  vaihtoehto lokitason mukauttamiseen, `servoshell`:lla on `--log-filter`-valinta ohos-kohteilla, joka mahdollistaa
  `log`-kirjaston lokisuodattimen mukauttamisen.
  Oletuksena servoshell asettaa lokisuodattimen, joka piilottaa lokiviestit monista crateista, joten todennäköisesti joudut
  asettamaan mukautetun log-filterin, jos et näe debuggaamasi craten lokeja.
3. `hilog`-palvelun peruslokisuodatin. `hdc shell hilog --base-level=<log_level>`. Voidaan yhdistää `--domain`- ja `--tag`-
  valintoihin mukauttaaksesi **mitkä lokit tallennetaan**.
4. `hilog`-palvelun lokisuodatin lokeja näytettäessä: `hdc shell hilog --level=<level>`

Useimmiten vaihtoehtoja 2 ja/tai 4 tulisi käyttää, koska ne mahdollistavat nopeat muutokset ilman uudelleenkääntämistä.

[log compile-time filters]: https://docs.rs/log/latest/log/#compile-time-filters

### Hilog-domainit

`hilog` mahdollistaa mukautetun kokonaisluvun „domain” (välillä 0–0xFFFF) asettamisen lokittaessa, mikä helpottaa lokien suodattamista domainin mukaan.
Servon Rust-koodissa, joka lokataan `log`-kirjaston kautta, käytämme domainia [`0xE0C3`] ja Spidermonkeyn C++-koodissa domainia [`0xE0C4`].
Nämä arvot on valittu melko mielivaltaisesti.

[`0xE0C3`]: https://github.com/servo/servo/blob/384d8f1ff895f070397b7a5a384428b1678416c5/ports/servoshell/egl/ohos.rs#L420
[`0xE0C4`]: <TODO - link pending SM PR being merged.>

### Hilog-yksityisyysominaisuus

`hilog`:lla on yksityisyysominaisuus, joka oletuksena piilottaa arvot lokeissa (esim. `%d`- tai `%s`-korvausten kohdalla).
Rustin lokiviestit eivät yleensä ole tämän vaikutuksen alaisia, koska merkkijonojen muotoilu tehdään Rust-puolella.
Jos kohtaat tämän ongelman C/C++-lokeja katsellessa, voit poistaa yksityisyysominaisuuden väliaikaisesti käytöstä ajamalla:

```shell
hdc shell hilog -p off
```

### DevTools ja porttien välitys

Voit ottaa DevTools-palvelimen käyttöön ja yhdistää siihen etänä Firefoxilla. Helpoin tapa on komentorivi:
`hdc shell aa start -a EntryAbility -b org.servo.servo --psn=--devtools=6080`
Yhdistääksesi Servo-instanssiin sinun täytyy välittää portti komennolla
`hdc fport tcp:6080 tcp:6080`. Sinun pitäisi nähdä viesti, että välitys onnistui. Nyt voit
yhdistää DevTools-palvelimeen osoitteella `localhost:6080`.
