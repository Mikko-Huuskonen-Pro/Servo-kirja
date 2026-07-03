<!-- TODO: needs copyediting -->

# Testaus

Tämä on tylsää.
Mutta PR:ääsi ei hyväksytä ilman testiä.

Servossa on kolmea testityyppiä.
- Koodin formatointitestit (`./mach test-tidy`).
- Yksikkötestit (`./mach test-unit`).
- Integraatiotestit (`./mach test-wpt`).

Tämän dokumentin painopiste on integraatiotesteissä, mutta haluamme mainita muut testit ensin.

Jokaisen lähetyksen on täytettävä koodin formatointitestit. Suurin osa formatoinnista voidaan tehdä
automaattisesti komennolla `./mach fmt`.

Yksikkötestit ovat eri tiedostoissa koko koodikannassa tyypillisillä rust `#[cfg(test)]`- ja
`#[test]`-annotaatioilla. Yksikkötestit ajetaan komennolla `./mach test-unit`. Esimerkkikutsuja:

```
# Run all unit tests in the net crate
./mach test-unit -p servo-net
# Run a specific unit test in the net crate
./mach test-unit -p servo-net test_fetch_response_is_not_network_error
```

Koko testisarjan ajaminen voi olla hyvin muistia kuluttavaa; voit lieventää tätä käyttäytymistä
rajoittamalla käännettävien cratejen määrää komennolla `./mach test-unit -j 4`.


## Integraatiotestit
Testit sijaitsevat `tests`-hakemistossa.
Huomaat, että siellä on paljon tiedostoja, joten oikean sijainnin löytäminen testillesi ei aina ole ilmeistä.

Katso ensin "Testing"-osio komennossa `./mach --help` ymmärtääksesi eri testikategoriat.
Löydät myös joitain `update-*`-komentoja.
Niitä käytetään odotettujen tulosten listan päivittämiseen.

Testin ajaminen:

```
./mach test-wpt tests/wpt/yourtest
```

## Uuden testin lisääminen

Jos sinun on luotava uusi testitiedosto, sen pitäisi sijaita hakemistossa `tests/wpt/mozilla/tests` tai hakemistossa `tests/wpt/tests`, jos se ei riipu vain Servo-ominaisuuksista.
Sinun on sitten päivitettävä testien lista ja odotettujen tulosten lista:

```
./mach test-wpt --manifest-update
```

## Testin debuggaus

Katso [debuggausopas](debugging/index.md) aloittaaksesi Servon debuggauksen.

## Web Platform Tests (`tests/wpt`)

Tämä kansio sisältää Web Platform Tests -testit ja koodin, jota tarvitaan niiden integroimiseen Servoon.
Lisäksi on WebGPU- ja WebGL-testejä, jotka on tuotu ja muunnettu Web Platform Test -tyylisiksi testeiksi.

### `tests/wpt`-hakemiston sisältö

Erityisesti tämä kansio sisältää:

* `config.ini`: konfiguraatio Web Platform Tests -testeille
* `include.ini`: Web Platform Tests -testien osajoukko, jota ajamme tällä hetkellä
* `tests`: Web Platform Tests -testien kopio repossa
* `meta`: odotetut epäonnistumiset ajamillemme Web Platform Tests -testeille
* `mozilla`: Web Platform Test -tyylisiä testejä, joita ei voi upstreamata
* `webgl`: tuodut WebGL-testit
* `webgpu`: tuodut WebGPU-testit (katso [WebGPU](../architecture/webgpu.md) -luku lisätietoja varten)

## Web Platform Tests -testien ajaminen

Yksinkertaisin tapa ajaa Web Platform Tests -testit Servossa on `./mach test-wpt` juurihakemistossa.
Tämä ajaa `include.ini`-tiedostossa määritellyn JavaScript-testien osajoukon ja kirjaa tulosteen stdoutiin.

Testien osajoukkoa voi ajaa antamalla positionaalisia argumentteja mach-komennolle joko tiedostojärjestelmäpolkuina tai testi-URL:ina, esim.

    ./mach test-wpt tests/wpt/tests/dom/historical.html

ajaaksesi testin dom/historical.html, tai

    ./mach test-wpt dom

ajaaksesi kaikki DOM-testit.

Test harness hyväksyy myös suuren määrän komentorivivalitsimia; ne on dokumentoitu ajamalla `--help`.

WPT-testien ajaminen debug-käännöksellä johtaa usein timeouteihin.
Sen sijaan harkitse kääntämistä komennolla `mach build -r` ja testaamista komennolla `mach test-wpt -r`.

### Web Platform Tests -testien ajaminen GitHub-forkissasi

Vaihtoehtoisesti voit suorittaa testit GitHubin isännöimillä runnereilla komennolla `mach try`.
Yleensä `mach try linux-wpt` (kaikki testit, linux) riittää.

Voit tarkastella ajotuloksia forkissasi "Actions"-välilehdeltä.
Epäonnistuneet tehtävät sisältävät listan vakaista odottamattomista tuloksista lokin lopussa.
Odottamattomat tulokset, jotka ovat tunnetusti intermittentejä, voidaan todennäköisesti jättää huomiotta.

Kun avaat PR:n, voit sisällyttää linkin ajoon. Muuten arvioijat ajavat testit uudelleen.

## Web Platform Test -odotusten päivittäminen

Kun korjaat bugin, joka muuttaa testin tulosta, kyseisen testin odotetut tulokset on muutettava.
Tämä voidaan tehdä manuaalisesti muokkaamalla `.ini`-tiedostoa `meta`-kansiossa, joka vastaa testiä.
Poista tässä tapauksessa viittaukset testeihin, joiden odotus on nyt `PASS`, ja poista `.ini`-tiedostot, jotka eivät enää sisällä odotuksia.

Kun tarvitaan suurempi määrä muutoksia, prosessi voidaan automatisoida:

    ./mach test-wpt --update-expectations path/to/tests/

Voit myös päivittää testiodotukset CI:llä suoritetuista Try-ajoista. Jokainen CI-ajo tallentaa automaattisesti `.log`-tiedoston, jonka voit antaa `update-wpt`-komennolle:

    ./mach update-wpt https://github.com/servo/servo/actions/runs/<ID-OF-CI-RUN>

*Kaikkien* Web Platform Tests -testien ajaminen paikallisesti kestää kauan ja aiheuttaa usein liittymättömiä epäonnistumisia (kuten runnerin ylittäessä järjestelmän maksimimäärän avoimia tiedostoja).
Testiodotukset asetetaan myös Servon CI-koneiden tulosten perusteella, joten ympäristösi erot voivat aiheuttaa epäonnistumisia.

Sinulla on yleensä karkea käsitys siitä, missä muutoksiisi liittyvät testit ovat.
Esimerkiksi lähes kaikki [SubtleCrypto](https://github.com/servo/servo/blob/63793ccbb7c0768af3f31c274df70625abacb508/components/script/dom/subtlecrypto.rs) -koodin testit ovat [`WebCryptoAPI`](https://github.com/web-platform-tests/wpt/tree/550fb109615cf434b03b30b76aa0dea6bfb0ebe1/WebCryptoAPI) -hakemistossa.
Tässä tapauksessa voit ajaa vain nämä testit komennolla `./mach test-wpt WebCryptoAPI`, ja sitten `./mach update-wpt` kuten yllä kuvataan.
Varmistaaksesi, ettei muita testejä rikkoutunut, tee sen jälkeen [try run](#running-web-platform-tests-on-your-github-fork).

WebGPU-odotusten päivittämiseen katso [WebGPU](../architecture/webgpu.md#updating-webgpu-cts-expectations) -luku.

## Web Platform Tests -testien muokkaaminen

Katso Web Platform Test -testien [Writing Tests guide](https://web-platform-tests.org/writing-tests/index.html) testien kirjoittamiseen.
Yleisesti hyvä tapa aloittaa on seurata näitä vaiheita:

1. Etsi hakemisto, joka testaa haluamaasi ominaisuutta.
2. Etsi hakemistosta testi, joka testaa samankaltaista käyttäytymistä, kopioi testi sopivalla nimellä ja tee muutoksesi.
3. Jos testi on reference test, kopioi myös referenssi, jos sen pitää muuttua, seuraten muokkaamasi hakemiston nimeämiskäytäntöä.
4. Aja `./mach update-manifest` päivittääksesi WPT-manifestin uusilla testeillä.

Sinun on myös ajettava `./mach update-manifest` aina, kun muokkaat testiä, referenssiä tai tukitiedostoa.
Kaikki muutokset repossa oleviin Web Platform Tests -testeihin upstreamataan automaattisesti, kun PR:si mergeataan.

## Servo-spesifiset testit

`mozilla`-hakemisto sisältää testejä, joita ei voi upstreamata jostain syystä (esim. koska ne riippuvat Servo-spesifisistä API:sta), sekä joitain legacy-testejä, jotka pitäisi upstreamata jossain vaiheessa.
Kun ne ajetaan, ne mountataan palvelimelle polkuun `/_mozilla/`.

## Reftest-tulosten analysointi

Reftest-tuloksia voi analysoida raakalokitiedostosta.
Luo se ajamalla `--log-raw`-valitsimella, esim.

```
./mach test-wpt --log-raw wpt.log
```

<!-- TODO: reftest analyzer link is dead -->
Tämän tiedoston voi syöttää [reftest analyzer](https://hg.mozilla.org/mozilla-central/raw-file/tip/layout/tools/reftest/reftest-analyzer-structured.xhtml) -työkalulle, joka näyttää kaikki epäonnistuneet testit (ei vain niitä, joilla on odottamaton tulos).
Huomaa, että tämä syö lokit eri muodossa kuin [työkalun alkuperäinen versio](https://hg.mozilla.org/mozilla-central/raw-file/tip/layout/tools/reftest/reftest-analyzer.xhtml), joka on kirjoitettu gecko-reftesteille.

Reftest analyzer mahdollistaa testin ja referenssin kuvakaappausten vertailun pikselitasolla.
Testit, jotka sekä epäonnistuvat että joilla on odottamaton tulos, on merkitty `!`-merkillä.

## WPT-manifestin päivittäminen

MANIFEST.json voidaan regeneroida automaattisesti mach-komennolla `update-manifest`, esim.

    ./mach update-manifest

Tämä on ekvivalentti ajolle

    ./mach test-wpt --manifest-update SKIP_TESTS

## DevTools-testien ajaminen

Yksinkertaisin tapa ajaa DevTools-testit Servossa on `./mach test-devtools` juurihakemistosta.
Tämä ajaa kaikki `devtools_tests/test_*.py`-tiedostoissa määritellyt testit. Kaikki DevTools-liittyvät testitiedostot ovat hakemistossa `python/servo/devtools_tests`.

## Vaihtoehtoja web platform tests -testien ajamiseen

Web platform tests -testejä voi ajaa myös muilla tavoilla.
Näitä ei suositella tyypillisissä kehitystyönkuluissa, koska ne vaativat mukautetun asennuksen.
Joskus näitä ajoja tarvitaan kuitenkin monimutkaisten vuorovaikutusten debuggaamiseen.

### Web Platform Tests -testien ajaminen ulkoisella palvelimella

Normaalisti wptrunner käynnistää oman WPT-palvelimensa, mutta joskus saatat haluta ajaa useita `mach test-wpt` -instansseja, esimerkiksi debugatessasi yhtä testiä samalla kun ajat koko sarjaa taustalla, tai kun ajat yhtä testiä monta kertaa rinnakkain (--processes toimii vain eri testien välillä).

Tämä johtaisi "Failed to start HTTP server" -virheisiin, koska vain yksi WPT-palvelin voi olla kerrallaan käynnissä.
Korjaa näin:

1. Seuraa vaiheita kohdassa [**Web-testien manuaalinen ajaminen**](#running-web-tests-manually)
2. Lisää `break` kohtaan [start_servers in serve.py](https://github.com/servo/servo/blob/01a9b317d4a6710547b8b0c0c476cc3b82251044/tests/wpt/tests/tools/serve/serve.py#L979-L1017) seuraavasti:
  ```
  --- a/tests/wpt/tests/tools/serve/serve.py
  +++ b/tests/wpt/tests/tools/serve/serve.py
  @@ -746,6 +746,7 @@ def start_servers(logger, host, ports, paths, routes, bind_address, config,
                     mp_context, log_handlers, **kwargs):
       servers = defaultdict(list)
       for scheme, ports in ports.items():
  +        break
           assert len(ports) == {"http": 2, "https": 2}.get(scheme, 1)

           # If trying to start HTTP/2.0 server, check compatibility
  ```
3. Aja `mach test-wpt` niin monta kertaa kuin tarvitset

Jos saat odottamattomia TIMEOUT-virheitä testharness-testeissä, custom testharnessreport.js on ehkä asennettu väärin (katso [**Web-testien manuaalinen ajaminen**](#running-web-tests-manually) lisätietoja varten).


### Web Platform Tests -testien manuaalinen ajaminen

(Katso myös [upstream README:n relevantti osio][upstream-running].)

On hyödyllistä ajaa testi ilman test runnerin häiriötä, esimerkiksi käyttäessä debuggeria kuten `gdb`.
Tätä varten WPT-palvelin on käynnistettävä manuaalisesti, mikä vaatii lisäkonfiguraatiota.

Lisää ensin seuraava järjestelmän hosts-tiedostoon:

    127.0.0.1   www.web-platform.test
    127.0.0.1   www1.web-platform.test
    127.0.0.1   www2.web-platform.test
    127.0.0.1   web-platform.test
    127.0.0.1   xn--n8j6ds53lwwkrqhv28a.web-platform.test
    127.0.0.1   xn--lve-6lad.web-platform.test

Siirry hakemistoon `tests/wpt/web-platform-tests` tämän osion loppuosaa varten.

Normaalisti wptrunner [asentaa Servon version testharnessreport.js:stä][environment], mutta kun WPT-palvelin käynnistetään manuaalisesti, saamme oletusversion, joka ei raportoi testituloksia oikein.
Korjaa näin:

1. Luo hakemisto `local-resources`
2. Kopioi `tools/wptrunner/wptrunner/testharnessreport-servo.js` → `local-resources/testharnessreport.js`
3. Muokkaa `local-resources/testharnessreport.js` korvaten muuttujat seuraavasti:
  * `%(output)d`
    * → `1` jos haluat leikkiä testillä interaktiivisesti (≈ pause-after-test)
    * → `0` jos et välitä siitä (vaikka `1` on ok aina)
  * `%(debug)s` → `true`
4. Luo `./config.json` seuraavasti (katso `tools/wave/config.default.json` oletuksia varten):
  ```
  {"aliases": [{
      "url-path": "/resources/testharnessreport.js",
      "local-dir": "local-resources"
  }]}
  ```

[environment]: https://github.com/servo/servo/blob/01a9b317d4a6710547b8b0c0c476cc3b82251044/tests/wpt/tests/tools/wptrunner/wptrunner/environment.py#L249-L257

Käynnistä palvelin komennolla `./wpt serve`.
Tarkistaaksesi, asennettiinko `testharnessreport.js` oikein:

* Komennon `curl http://web-platform.test:8000/resources/testharnessreport.js` standarditulosteen pitäisi näyttää [testharnessreport-servo.js]:ltä, ei [the default testharnessreport.js]:ltä
* Komennon `target/release/servo http://web-platform.test:8000/css/css-pseudo/highlight-pseudos-computed.html` standardituloste (tai mikä tahansa [testharness test]) pitäisi sisältää rivit, jotka alkavat:
    * `TEST START`
    * `TEST STEP`
    * `TEST DONE`
    * `ALERT: RESULT:`

[testharnessreport-servo.js]: https://github.com/servo/servo/blob/01a9b317d4a6710547b8b0c0c476cc3b82251044/tests/wpt/tests/tools/wptrunner/wptrunner/testharnessreport-servo.js
[the default testharnessreport.js]: https://github.com/servo/servo/blob/01a9b317d4a6710547b8b0c0c476cc3b82251044/tests/wpt/tests/tools/wptrunner/wptrunner/testharnessreport.js
[testharness test]: http://web-platform-tests.org/writing-tests/testharness.html

Estääksesi selaimen SSL-varoitukset HTTPS-testejä ajaessa paikallisesti, sinun on ajettava Servo valitsimella `--certificate-path resources/cert-wpt-only`.

[upstream-running]: https://github.com/w3c/web-platform-tests#running-the-tests

### Web Platform Tests -testien ajaminen Firefoxissa

Testien parissa työskennellessä saatat haluta vertailla Servon tulosta Firefoxiin.
Voit antaa `--product firefox` sekä polun Firefox-binääriin (sekä muutamia muita asioita) ajaaaksesi testejä Firefoxissa Servo-checkoutistasi:

    GECKO="$HOME/projects/mozilla/gecko"
    GECKO_BINS="$GECKO/obj-firefox-release-artifact/dist/Nightly.app/Contents/MacOS"
    ./mach test-wpt dom --product firefox --binary $GECKO_BINS/firefox --certutil-binary $GECKO_BINS/certutil --prefs-root $GECKO/testing/profiles
