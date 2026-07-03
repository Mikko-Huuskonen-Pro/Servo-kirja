# Satunnaisten WPT-virheiden diagnosointi

Yleisimmät satunnaisten epäonnistumisten lähteet Web Platform Testeissä ovat:

* Ajoitus (esim. useita säikeitä ajetaan samanaikaisesti tai rinnakkain)
* Käyttöjärjestelmävuorovaikutukset (esim. socket read/write verkkopyynnöissä)
* Ei-deterministinen tehtävän valinta

Jokainen näistä voi muuttaa tapahtumien järjestystä Servossa ei-deterministisesti, paljastaen odottamatonta tai suunnittelematonta käyttäytymistä moottorissa.

Servossa tämä ei-determinismi ilmenee usein koodissa, joka:

* käyttää mutexeja tai jaettua muistia kanavien sijaan.
* jonottaa useita tehtäviä, jotka ajetaan yhdellä säikeellä mutta eri task source -lähteistä.
* käyttää timeouteja havaitakseen, tapahtuiko tapahtuma.

## Epäonnistumisen toistaminen

Vaikka tietylle satunnaiselle epäonnistumiselle voi keksiä teorian, on hyödyllistä näyttää ennen/jälkeen-epäonnistumisprosentti yritykselle tehdä korjaus.

Strategioita satunnaisten epäonnistumisten toistamiseen:
* Aja `--repeat-until-unexpected`-valinnalla toistaaksesi testin, kunnes raportoidaan odottamaton tulos
* Aja testi suuren kuormituksen alla (esim. aja puhdas release-käännös toisessa terminaalissa testiä toistuvasti ajaessasi)
* Kokeile eri käännös- tai ajonaikaisia konfiguraatioita:
  * `./mach build --dev`
  * `./mach build --release`
  * `./mach build --debug-mozjs`
  * `./mach test-wpt ... --binary-args=--force-ipc`
  * `./mach test-wpt ... --binary-args=--multiprocess`

Varmista, että `test-wpt`-komentosi käyttää samaa ajonaikaista binääriä kuin tekemäsi käännös!

## Ongelman diagnosointi

Kun olet toistanut epäonnistumisen, sinun täytyy selvittää, mikä eroaa ajoissa, jotka raportoivat eri tuloksia.
Testatessasi muutoksia pidä mielessä satunnaisten tulosten esiintymistiheys.
Älä hyppää johtopäätöksiin odottamatta tarpeeksi kauan!

Aloita kommentoimalla pois niin paljon testistä kuin voit ja silti toistaen epäonnistumisen.

Kokeile lisätä `console.log`-kutsuja näyttääksesi eri tapahtumien järjestyksen, ja vertaa tulostetta yleiseen tapaukseen ja satunnaiseen tapaukseen.
Saatat joutua [asettamaan `RUST_LOG`-ympäristömuuttujan](../../debugging/index.md#debug-lokitus-log-kirjastolla-ja-rust_log-muuttujalla) nähdäksesi relevanttien cratejen ja moduulien sisäiset Servo-jäljitet lokit.

Lisää eksplisiittisiä viiveitä testin osiin havaitaksesi, tapahtuvatko epäonnistumiset todennäköisemmin vai harvemmin:
* siirrä osa testistä closureen, joka suoritetaan kiinteän viiveen jälkeen, kuten `test_object.step_timeout(() => ..., 1000)`
* [lisää viive](https://web-platform-tests.org/writing-tests/server-pipes.html#trickle) verkkopyynnön vastaukseen

## Layout-testien ongelmien diagnosointi

Jos testi varmistaa layout-ominaisuuksia (joko reftest tai testi, joka käyttää layout-API:ja kuten `getBoundingClientRect()`, `scrollTop`, `offsetParent` jne.), yleisiä satunnaisten tulosten lähteitä ovat:

1. inkrementaalinen layout toimii väärin tietylle muutokselle, mutta tämä peittyy toisella async-operaatiolla, joka laukaisee lisälayoutin
2. kuvakaappaus otetaan liian aikaisin/myöhään suhteessa johonkin muuhun muutokseen (esim. web-fontin lataus)

Tehdäksesi inkrementaalisen layoutin ongelmista näkyvämpiä, kokeile:
* viivästää sivun muokkausta, kunnes kaikki muut sivupäivitykset on valmis (kokeile hyvin viivästettyä `setTimeout`-kutsua).
* ajaa testi oikealla ikkunalla (`--no-headless`).
* olla koskematta hiireen, kunnes sivun muokkaus tapahtuu, ja sitten muuttaa ikkunan kokoa.

Selvittääksesi, onko kuvakaappauksen ajoitus ongelma, käytä [reftest-analysaattoria](../../testing.md#analyzing-reftest-results) nähdäksesi testikehyksen vastaanottaman kuvakaappauksen.
Jos se ei vastaa tulosta, jonka näet [testitiedostoa ajaessasi](stable-wpt-errors.md#reftest-epaonnistumisen-diagnosointi), kuvakaappauksen ajoitus voi olla syypää.
