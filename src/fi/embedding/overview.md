# Yleiskuva: Servon upottaminen

Servo on selainmoottori, jonka on tarkoitus olla helppo upottaa muihin sovelluksiin.
Tällä hetkellä dokumentaatio Servon upottamisesta on niukkaa, ja tämä luku on aktiivisesti työn alla.
Ota rohkeasti yhteyttä [Servo Zulip] -keskustelussa, jos sinulla on kysyttävää.

[Servo Zulip]: https://servo.zulipchat.com

## Servon kääntäminen ilman machia

Servo voidaan kääntää suoraan `cargo`:lla, vaikka joudut todennäköisesti toteuttamaan osan machin toiminnallisuudesta uudelleen.
Alla on lyhyt yleiskuva asioista, joita kannattaa pitää mielessä.


### Ympäristömuuttujat

`./mach` asettaa useita ympäristömuuttujia käännöksen ohjaamiseen, mikä on erityisen tarpeellista ristiinkäännöksessä esim. Androidille tai OpenHarmonylle.
Voit ajaa `./mach print-env` nähdäksesi mitkä muuttujat asetetaan, ja käyttää sitä käännöksen konfigurointiin.

### Mediatuki

`./mach build` ottaa `media-gstreamer` -ominaisuuden oletuksena käyttöön, mikä ei päde tavallisiin käännöksiin.
`media-gstreamer` vaaditaan mediatuelle yleisillä työpöytäalustoilla. 
Linuxilla sinun täytyy asentaa vaaditut `gstreamer`-kirjastot paketinhallinnan kautta (esim. ajamalla ./mach bootstrap).
macOS:lla `./mach bootstrap` asentaa vaaditut `gstreamer`-kirjastot asentamalla uusimmat paketit osoitteesta https://github.com/servo/servo-build-deps/releases/tag/macOS.
Windowsilla voit asentaa gstreamer-kirjastot osoitteesta https://github.com/servo/servo-build-deps/releases/tag/msvc-deps (versio 1.22.8).

### Resurssit

Servo tarvitsee resursseja ja tarjoaa oletusversiot `servo-default-resources` -craten kautta, jos `baked-in-resources` -ominaisuus on käytössä.
Tämä toteutetaan upottamalla resurssit binääriin. 
Upottajat voivat kieltäytyä tästä poistamalla default-features -ominaisuudet käytöstä ja tarjoamalla oman resurssienlukumekanismin [servo_embedder_traits::submit_resource_reader]:n kautta.
Tällöin ResourceReader-toteutuksen vastuulla on tarjota kaikki resurssit.

[servo_embedder_traits::submit_resource_reader]: https://docs.rs/servo-embedder-traits/latest/embedder_traits/macro.submit_resource_reader.html

Esimerkki on OpenHarmony-portti, joka lukee kaikki resurssit tiedostojärjestelmästä: [ohos/resources.rs](https://github.com/servo/servo/blob/78c9fe2a4c7a645e7ec729094bfe3eb8a6c15189/ports/servoshell/egl/ohos/resources.rs).
