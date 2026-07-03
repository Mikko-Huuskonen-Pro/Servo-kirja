# Vakaiden WPT-virheiden diagnosointi

## FAIL-tuloksen diagnosointi

Aloita avaamalla testitiedoston `.ini`-tiedosto, joka sisältää odotetut epäonnistumiset.
Se löytyy hakemistosta `tests/wpt/meta/` rinnakkaisessa hakemistopuussa suhteessa `tests/wpt/tests/`-hakemistoon.

WPT-testikehys suppressoi suurimman osan `.ini`-tiedostossa odotetuiksi merkittyihin epäonnistumisiin liittyvästä tulosteesta.

Nähdäksesi kaikki testitiedoston epäonnistumiset, muuta ylätason `[filename.html]`-kohtaa niin, ettei se vastaa todellista tiedostonimeä; tämä saa kehyksen jättämään `.ini`-tiedoston kokonaan huomiotta.

Nähdäksesi lisätietoja yhdestä alitestistä, poista se `.ini`-tiedostosta ja aja testi uudelleen.
Kehys näyttää epäonnistuvat testiväitteet sekä JS-stack tracen, kun epäonnistumiset tapahtuvat.

## ERROR-tuloksen diagnosointi

ERROR-tulokset syntyvät, kun poikkeus heitetään ilman että sitä käsitellään.
Testikehyksen stack tracen pitäisi näyttää alitesti, jossa käsittelemätön poikkeus havaittiin, sekä virheen tyyppi.

Jos poikkeus tulee Rustissa toteutetun API-metodin kutsumisesta, sinun täytyy löytää kyseinen metoditoteutus ja etsiä koodia, joka palauttaa vastaavan [`Error`](https://doc.servo.org/script/dom/bindings/error/enum.Error.html)-variantin.

## TIMEOUT-tuloksen diagnosointi

Testien timeout tapahtuu, kun async/promise-alitesti on määritelty mutta ei koskaan valmistu.
Sinun täytyy tunnistaa kaksi asiaa:

1. viimeinen testikoodi, joka suoritetaan onnistuneesti
2. miksi seuraavaa koodia, joka pitäisi suorittaa, ei koskaan ajeta

Nopein tapa selvittää ensimmäinen tietopiste on lisätä `console.log`-lauseita todistamaan, että koodipolut suoritetaan.

Koodi, jota ei suoriteta, on yleensä:
* tapahtumakäsittelijä (joko tapahtumaa ei koskaan laukaista, tai käsittelijässä on suodatuslogiikka, joka aktivoituu)
* promise-käsittelijä (promisea ei koskaan resolvata/rejectata)
* await-lause (promisea ei koskaan resolvata/rejectata)

Kussakin tapauksessa on hyödyllistä aloittaa koodista, jonka on tarkoitus laukaista puuttuva vaihe (esim. relevantti `event.fire(..)` Rust-koodissa), ja työskennellä taaksepäin selvittääksesi, miksi sitä ei koskaan suoriteta.

## NOTRUN-tuloksen diagnosointi

Nämä epäonnistumiset tapahtuvat, kun async-alitesti on määritelty (esim. `let some_test = async_test("frobbing the whatsit");`) mutta sitä ei koskaan suoriteta (esim. `some_test.step(() => ...)`).
Tämä tapahtuu yleensä, kun testitiedosto määrittää monta alitestiä ja yrittää ajaa ne peräkkäin, mutta kohtaa poikkeuksen tai timeoutin, joka estää jatkosuorituksen.

Nämä tulokset ovat yleensä oire jostain muusta ongelmasta, jonka testitiedosto paljastaa, ja näitä muita ongelmia pitäisi tutkia ensin.

## Reftest-epäonnistumisen diagnosointi

Nähdäksesi visuaalisen esityksen eroista testitiedoston ja sen referenssitiedoston välillä, voit käyttää [reftest-analysaattoria](../../testing.md#analyzing-reftest-results).

Voit nopeasti tarkastella reftestin ulkoasua ajamalla tiedoston suoraan (`./mach run tests/wpt/tests/css/CSS2/some-file.html`).

Jos tiedosto riippuu muista resursseista testisuitesta, sinun täytyy ehkä käynnistää WPT-web-palvelin ensin:
* `cd tests/wpt/tests; ./wpt serve`
* `./mach run http://localhost:8000/css/CSS2/some-file.html`

## Debuggerin käyttö Web Platform Testeissä

Liittääksesi debuggerin Servoon WPT-testiä ajaessasi, lisää `--debugger`-lippu `./mach test-wpt`-komentoon.
On kaksi toimintatapaa:

1. Kun käytät `servodriver`-testikehystä (oletus), sinun täytyy ajaa komento debuggerin liittämiseksi toisessa terminaalissa.
Esimerkkikomento annetaan `./mach test-wpt`-komennon tulosteessa; testikehys odottaa sitten 30 sekuntia ennen suorituksen jatkamista.
2. Kun käytät `servo`-testikehystä (`--product=servo`), debugger liitetään automaattisesti.
Jatkaaksesi testikehyksen suoritusta, käytä debuggerin kehotteessa `run`-komentoa.
