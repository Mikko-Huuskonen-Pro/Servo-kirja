<!-- TODO: needs copyediting -->

# Profilointi

Kun profiloit Servoa tai vianmetsästät suorituskykyongelmia, varmista, että käännöksesi on optimoitu mutta silti sallii tarkan profilointidatan.

```sh
./mach build --profile profiling --with-frame-pointer
```

- **--profile profiling** kääntää Servon [profilointikonfiguraatiollamme](../building/building.md#build-profiles)
- **--with-frame-pointer** kääntää Servon pinot-kehyspointtereilla kaikilla alustoilla

Vastaava profiili on valittava Servoa ajettaessa:

```sh
./mach run --profile profiling http://example.org
```

Useita tapoja saada profilointitietoa Servon ajoista:
* [Tracing Perfetton kanssa](#tracing-with-perfetto)
* [Interval Profiling](#interval-profiling)
  * [TSV Profiling](#tsv-profiling)
  * [Generating Timelines](#generating-timelines)
  * [Built-in sampling profiler](#sampling-profiler)
* [Memory Profiling](#memory-profiling)
* [Using macOS Instruments](#using-macos-instruments)


## Tracing Perfetton kanssa

Tracing toimii instrumentoimalla tiettyjä funktioita (tai koodin osia) eksplisiittisillä annotaatioilla kuten `time_profile!` ja `servo_tracing::*`.
Se tallentaa deterministisesti jokaisen kutsun instrumentoidun koodin läpi, mutta ei tarjoa näkyvyyttä koodiin, jota ei ole instrumentoitu.

Sitä vastoin sampling-profilerit näkevät kaiken, mutta vain probabilistisesti, ja tarkentuvat siten pidemmillä silmukoilla.

Tracingin käyttöön ota käyttöön liittyvät compile-time-ominaisuudet:

```sh
./mach build --profile profiling --features tracing tracing-perfetto
```

Aja sitten Servo `SERVO_TRACING`-ympäristömuuttujalla, joka on asetettu [`EnvFilter` directives] -direktiiveihin valitaksesi, mitkä trace:t otetaan käyttöön:

```
SERVO_TRACING=… ./mach run --profile profiling http://example.org
```

Esimerkiksi:

- `SERVO_TRACING=off` poistaa kaiken tracingin käytöstä (tämä on oletus)
- `SERVO_TRACING=trace` ottaa kaiken tracingin käyttöön (tuottaa valtavia trace-tiedostoja `compositing`-moduulin vuoksi)
- `SERVO_TRACING=[{servo_profiling}]` tekee saman, koska suodatamme implisiittisesti `servo_profiling`-arvolla
- `SERVO_TRACING=info` ottaisi käyttöön vain `info`-tason ja yläpuolella olevat, mutta emme vielä käytä tasoja
- `SERVO_TRACING=layout` ottaa tracingin käyttöön vain `layout`-cratessa
- `SERVO_TRACING=trace,compositing=off` ottaa kaiken tracingin käyttöön paitsi `compositing`-cratessa
- `SERVO_TRACING=[handle_reflow]` ottaa tracingin käyttöön spaneissa nimeltä "handle_reflow" *tai niiden jälkeläisissä*

Tämä luo `servo.pftrace`-tiedoston nykyiseen hakemistoon, jota voi visualisoida osoitteessa [ui.perfetto.dev](https://ui.perfetto.dev/).

[`EnvFilter` directives]: https://docs.rs/tracing-subscriber/0.3.23/tracing_subscriber/filter/struct.EnvFilter.html#directives


## Interval Profiling

Käyttämällä `-p`-valitsinta ja sen jälkeen numeroa (aikajakso sekunteina) voit tulostaa profilointitietoa terminaaliin säännöllisesti.
Tätä varten aja Servo halutulla sivustolla (URL-osoitteet ja paikalliset tiedostopolut molemmat tuettuja) profilointi käytössä:

```
./mach run --profile profiling http://example.org -p 5
```

Yllä olevassa esimerkissä, kun Servo on yhä käynnissä (JA käsittelee uusia pass-vaiheita), profilointitieto tulostetaan terminaaliin 5 sekunnin välein.

Kun sivu on ladattu, paina ESC (tai sulje sovellus) poistuaksesi.
Profilointituloste tarjotaan jaoteltuna selainmoottorin alueisiin ja URL:ään.
Esimerkiksi saatat saada alla olevan muotoisen tulosteen:

```
_category_                          _incremental?_ _iframe?_             _url_                  _mean (ms)_   _median (ms)_      _min (ms)_      _max (ms)_       _events_ 
Painting                                N/A          N/A                  N/A                        6.8177          0.9512          0.0035         30.7573               6
Layout                                  yes          yes     http://example.org/                     0.0016          0.0016          0.0016          0.0016               1
Layout                                  no           yes     http://example.org/                    14.4966         14.4966         14.4966         14.4966               1
ScriptParseHTML                         no           yes     http://example.org/                     0.8507          1.7009          0.0004          1.7009               2
TimeToFirstPaint                        no           yes     http://example.org/                     0.0000          0.0000          0.0000          0.0000               1
TimeToFirstContentfulPaint              no           yes     http://example.org/                     0.0000          0.0000          0.0000          0.0000               1

_url_   _blocked layout queries_

```

Tässä esimerkissä sivun latauksessa suoritimme yhden täyden asettelun ja yhden inkrementaalisen asettelun.

### TSV Profiling

Käyttämällä `-p`-valitsinta ja sen jälkeen tiedostonimeä voit tulostaa Servon suorituksen profilointitiedot TSV-tiedostoon (tab-separated, koska tietyt url:t sisältävät pilkkuja).
Tiedot kirjoitetaan tiedostoon vasta Servon lopettamisen yhteydessä.
Tämä toimii hyvin `-x`-, `-z`- ja `-o`-valitsimien kanssa, jotta suorituskykytietoa voidaan kerätä automatisoiduissa ajoissa.
Esimerkkikäyttö:

```
./mach run --profile profiling http://example.org -zxo out.png -p out.tsv
```

Interval- ja TSV Profiling -valintojen profilointitiedon formaatit ovat käytännössä samat; url-nimiä ei katkaista TSV Profiling -valinnossa.

### Generating Timelines

Lisää `--profiler-trace-path /timeline/output/path.html` -lippu tulostaaksesi profilointidatan itse sisältävänä HTML-aikajanana.
Koska se on itse sisältävä tiedosto (kaikki CSS ja JS inline), sitä on helppo jakaa, ladata tai linkittää bugiraportteihin.

```
./mach run --profile profiling http://example.org -p 5 --profiler-trace-path trace.html
```

#### Usage:

* Käytä hiiren rullaa tai trackpadin scrollausta, hiiren ollessa aikajanan yläosassa, zoomataksesi näkymää sisään tai ulos.

* Tartu valittuun alueeseen yläosassa ja vedä vasemmalle tai oikealle sivusuuntaiseen scrollaukseen.

* Vie hiiri trace:n päälle nähdäksesi lisätietoja.

#### Hacking

Aikajanan JS, CSS ja HTML tulee reposta [fitzgen/servo-trace-dump](https://github.com/fitzgen/servo-trace-dump/), ja siellä on skripti Servon kopion päivittämiseen.

Kaikki muu koodi on hakemistossa `components/profile/`.

## Sampling profiler

Servo sisältää sampling-profilerin, joka tuottaa profiileja, jotka voidaan avata [Gecko profiling tools](https://profiler.firefox.com/) -työkaluissa.
Käyttö:

1. Aja Servo lataamalla profiloitava sivu
2. Paina Ctrl+P (tai Cmd+P macOS:ssa) käynnistääksesi profilerin (konsolin pitäisi näyttää "Enabling profiler")
3. Paina Ctrl+P (tai Cmd+P macOS:ssa) pysäyttääksesi profilerin (konsolin pitäisi näyttää "Stopping profiler")
4. Pidä Servo käynnissä, kunnes symbolien resoluutio on valmis (konsolin pitäisi näyttää lopuksi "Resolving N/N")
5. Aja `python etc/profilicate.py samples.json >gecko_samples.json` muuntaaksesi profiilin muotoon, jonka Gecko profiler ymmärtää
6. Lataa `gecko_samples.json` osoitteeseen https://profiler.firefox.com/

Tulostetiedoston nimen hallintaan aseta `PROFILE_OUTPUT`-ympäristömuuttuja.
Näytteenottotaajuuden hallintaan (oletus 10 ms) aseta `SAMPLING_RATE`-ympäristömuuttuja.

## Memory Profiling

* Aja Servoshell normaalisti
* Avaa uusi välilehti
* Navigoi osoitteeseen `about:memory`
* Napsauta `Measure`
* Näet raportin, joka muistuttaa tätä:

```
  115.15 MiB -- explicit
     101.15 MiB -- jemalloc-heap-unclassified
      14.00 MiB -- url(http://example.org/)
         10.01 MiB -- layout-thread
            10.00 MiB -- font-context
             0.00 MiB -- stylist
             0.00 MiB -- display-list
          4.00 MiB -- js
             2.75 MiB -- malloc-heap
             1.00 MiB -- gc-heap
                0.56 MiB -- decommitted
                0.35 MiB -- used
                0.06 MiB -- unused
                0.02 MiB -- admin
             0.25 MiB -- non-heap
       0.00 MiB -- memory-cache
          0.00 MiB -- private
          0.00 MiB -- public

  121.89 MiB -- jemalloc-heap-active
  111.16 MiB -- jemalloc-heap-allocated
  203.02 MiB -- jemalloc-heap-mapped
  272.61 MiB -- resident
```

## Using macOS Instruments

Xcodessa on [instruments](https://help.apple.com/instruments/mac/10.0/) -työkalu helppoon profilointiin.

Ensin asenna Xcode instruments:

```
xcode-select --install
```

Toiseksi asenna `cargo-instruments` Homebrewin kautta:

```
brew install cargo-instruments
```

Sitten voit ajaa sen suoraan CLI:stä:

```
cargo instruments -t Allocations
```

Tässä on linkkejä ja resursseja Instrumentsin käyttöön (jotkut streamaavat vain Safarissa):
* [cargo-instruments on crates.io](https://crates.io/crates/cargo-instruments)
* [Using Time Profiler in Instruments](https://developer.apple.com/videos/play/wwdc2016/418/)
* [Profiling in Depth](https://developer.apple.com/videos/play/wwdc2015/412/)
* [System Trace in Depth](https://developer.apple.com/videos/play/wwdc2016/411/)
  * Threads, virtual memory, and locking
* [Core Data Performance Optimization and Debugging](https://developer.apple.com/videos/play/wwdc2013/211/)
* [Learning Instruments](https://developer.apple.com/videos/play/wwdc2012/409/)


## Profiling WebRender

Kun ajat Servoshelliä, paina CTRL+F12 näyttääksesi (tai piilottaaksesi) WebRender-overlayn.


## Webpage snapshots

On mahdollista käyttää `mitmproxy`-työkalua Servon liikenteen sieppaamiseen ja luoda paikallinen snapshot (dump) mielivaltaisesta verkkosivusta, jota voidaan sitten tarjoilla paikallisesti profilointitarkoituksiin.

`mitmproxy` tukee useita tapoja siepata liikennettä, mukaan lukien `proxy`-tila portissa `:8080`, joten voit asettaa selaimen vain:

```bash
./target/release/servo \
--pref=network_http_proxy_uri=http://127.0.0.1:8080 \
--ignore-certificate-errors 
```

> [!info] The default `mitmproxy` certs are in the `~/.mitmproxy` or you can generate some using `mitmproxy`, but I have just set my browser to ignore cert errors

> [!warning] ignoring certs is easy, but be cautious of risks

### Default mitmproxy

Oletusverkossa mitmproxy luo paikallisen proxy-palvelimen porttiin `:8080`, ja asettamalla sen selaimessa tai antamalla `http_proxy=localhost:8080` ja/tai `https_proxy=localhost:8080` (ja valinnaisesti poistamalla `no_proxy`-asetuksen) voit dumppata ja tarjoilla liikennettä.

#### Creating a dump
```bash
mitmproxy -w <dumpfile>
```

#### Serving a dump
```bash
mitmproxy --server-replay <dumpfile>
```

Tuloksena oleva dump-tiedosto on noin `~5MB` per sivu, joten se kasvaa nopeasti suureksi, koska työkalu on hyvin verbose ja voi tallentaa kuvia.

### Chain-proxy
Toisen ensisijaisen proxy-yhteyden tapauksessa meidän on välitettävä `upstream` pääproxy:lle `mitmproxy`:sta, ja jos pääproxyllä on myös custom-sertifikaatteja, on tärkeää välittää ne tai jättää ne huomiotta
> [!warning] ignoring certs is easy, but be cautious of risks
#### Creating a dump
```bash
mitmproxy  --mode upstream:${http_proxy} -w <dump-path>\
--set ssl_insecure=true
#### Serving the dump
```bash
mitmproxy -v  --server-replay ~/dev/recodings/servo_org_3.dump \
--set server_replay_extra=404 \
--set server_replay_ignore_host=true \
--set connection_strategy=lazy \
--set server_replay_reuse=true
```
> [!info] the `replay_extra` and `replay_reuse` are optional, and may cause unexpected behaviour
### OpenHarmony
Työkalua voi käyttää etäpuhelimen liikenteen sieppaamiseen, mukaan lukien OpenHarmony-kohteet. Avaa käänteinen proxy-portti `hdc`:llä ja aja sitten `servo` proxy- ja sertifikaattiasetuksilla.
#### reverse port
```bash
hdc rport tcp:8080 tcp:8080
```
#### run with args
```bash
hdc shell aa start -a EntryAbility \
-b org.servo.servo -U https://servo.org \
--psn=--pref=network_http_proxy_uri=http://127.0.0.1:8080 \
--psn=--ignore-certificate-errors
```

