# Servodriver-testiharness

Servodriver on wptrunner-kehyksen ja WebDriver-palvelimen päälle rakennettu testiharness.
Se ei ole vielä oletuksena käytössä, mutta sen voi ajaa lisäämällä `--product servodriver` -argumentin mihin tahansa `test-wpt`-komentoon.
Servodriver koostuu kolmesta pääkomponentista: Python-testiharness, joka orkestroi selainta, Servon sisäinen web-palvelin, joka toteuttaa [WebDriver-spesifikaation](https://www.w3.org/TR/webdriver2/), ja testisivujen sisällä ladattavat skriptit.

## wptrunner-harness

Wptrunner on harness, joka käyttää `executor`- ja `browser`-yhdistelmää tuen tuotteen käynnistämiseen.
[browser](https://github.com/servo/servo/blob/main/tests/wpt/tests/tools/wptrunner/wptrunner/browsers/servodriver.py) määrittelee Python-luokat instanssien luomiseen ja metodit tuettujen konfiguraatioiden kutsumiseen sekä argumentit Servo-binääriä käynnistettäessä.
Se delegoi kaiken WebDriver-spesifisen logiikan perus [WebDriverBrowser-luokalle](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/browsers/base.py#L294), joka on yhteinen kaikille WebDriveriin luottaville selaimille.

Servodriverin [executor](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorservodriver.py) määrittelee Servo-spesifisen testien alustuksen, tulosten käsittelyn ja testien välisen tilan hallinnan.
Esimerkiksi Servo määrittelee [WebDriver-laajennusmetodit](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorservodriver.py#L23-L42) preferenssien hallintaan, ja näitä [kutsutaan](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorservodriver.py#L88-L92) testien välillä varmistaakseen, että jokainen testi ajaa [tarkoitetulla konfiguraatiollamme](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/meta/webxr/__dir__.ini#L1).

Servodriver-executormme delegoi paljon logiikkaa yhteiselle [WebDriverTestharnessExecutor](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L948) -luokalle.
Tämä executor on vastuussa [testdriver.js-harnessin](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L840) suorittamisesta ja mahdollistaa Python-harnessin hakevan testitulokset selaimesta.

## Servon WebDriver-toteutus

Tässä toteutuksessa on kolme pääkomponenttia: server handler, script handler ja input handler.

### Server handler

Kun wptrunner yhdistää [WebDriver-palvelimeen](https://github.com/servo/servo/tree/main/components/webdriver_server) uudessa Servo-instanssissa, se luo uuden [session](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/lib.rs#L132) -instanssin.
Tämä sessio säilyttää tilaa WebDriver API -kutsujen välillä.
Selaimen kanssa vuorovaikuttavat API-kutsut reititetään constellationin kautta [ConstellationMsg::WebDriverCommand](https://doc.servo.org/servo/enum.WebDriverCommandMsg.html) -viesteinä, ja ne, joiden täytyy vuorovaikuttaa suoraan dokumentin sisällön kanssa, kuuluvat [WebDriverScriptCommand](https://doc.servo.org/servo/enum.WebDriverScriptCommand.html) -enum:iin.
Monet näistä API:sta vaativat synkronisen vastauksen `IpcSender`-kanavan kautta, jota Servon WebDriver-palvelin käyttää [odottaakseen vastausta](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/lib.rs#L911).
Tapaukset kuten navigointi ja asynkronisten skriptien suoritus sisältävät kuitenkin lisäsynkronointia.

#### Navigointi

Kun navigointia pyydetään, [constellation vastaanottaa IpcSenderin](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/lib.rs#L677), jonka se tallentaa.
Kun navigointi valmistuu, constellation [tarkistaa, vastaako pipeline](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/constellation/constellation.rs#L3676) WebDriverin navigointia, ja ilmoittaa WebDriver-palvelimelle alkuperäistä kanavaa käyttäen.

#### Asynkroniset skriptit

Kun asynkronisen skriptin suoritus aloitetaan, WebDriver-spesifikaatio [tarjoaa tavan](https://www.w3.org/TR/webdriver2/#execute-async-script) skriptien kommunikoida tulosta palvelimelle.
Skripti kutsutaan [anonyyminä funktiona](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/lib.rs#L1532) [lisäfunktioargumentilla](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/lib.rs#L1524), joka lähettää vastauksen palvelimelle.

### Script handler

Kun WebDriver-viesti, joka kohdistuu tiettyyn selauskontekstiin, vastaanotetaan, ScriptThread, joka sisältää aktiivisen dokumentin, [käsittelee sen](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/script/script_thread.rs#L2076).
Kunkin komennon käsittelylogiikka sijaitsee [webdriver_handlers.rs](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/script/webdriver_handlers.rs) -tiedostossa.
Mikä tahansa komento, joka palauttaa web-sisällöstä johdetun arvon, täytyy [serialisoida JS-arvot](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/script/webdriver_handlers.rs#L162) arvoiksi, jotka WebDriver-palvelin voi muuntaa [API-arvotyypeiksi](https://doc.servo.org/serde_json/value/enum.Value.html).

Altistamme [kaksi webistä saavutettavaa metodia](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/script/dom/window.rs#L1189-L1203) asynkronisten tulosten kommunikointiin WebDriver-palvelimelle: `Window.webdriverCallback` ja `Window.webdriverTimeout`.

### Input handler

Kun WebDriver-palvelin vastaanottaa syöte-toimintokomentoja (esim. osoitin- tai hiiritoiminnot), se luo toimintosekvenssin ja [dispatch:aa ne asteittain](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/webdriver_server/actions.rs#L110) compositorille [constellationin kautta](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/constellation/constellation.rs#L4548-L4559).
Compositor [käsittelee nämä tapahtumat](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/components/compositing/compositor.rs#L587-L601) samalla tavalla kuin upottajalta vastaanotetut syöte-tapahtumat (poislukien [tunnetut bugit](https://github.com/servo/servo/issues/35394)).

## Testisivujen skriptit

testdriver-harness yhdistää kaikki nämä elementit yhteen.
Executor [avaa uuden ikkunan](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L850) ja kutsuu asynkronista skriptiä, joka [asettaa testdriver-callbackin](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/testharness_webdriver_resume.js) arvoon `Window.webdriverCallback` (`arguments[arguments.length - 1]`).
Tämä ikkuna luo [viestijonon](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/message-queue.js#L30), jota [executor-harness lukee](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L904-L905).
Myöhemmin jokainen ajettava testi avataan uutena ikkunana; nämä ikkunat voivat käyttää `opener.postMessage` kommunikoidakseen alkuperäisen ikkunan kanssa, mikä mahdollistaa testdriver API:en lähettää viestejä, jotka [edustavat WebDriver API -kutsuja](https://github.com/servo/servo/blob/3421185737deefe27e51e104708b02d9b3d4f4f3/tests/wpt/tests/tools/wptrunner/wptrunner/testdriver-extra.js#L286-L296).

## Debuggausvinkit

Aloita aina seuraavalla RUST_LOG:lla:

```
RUST_LOG=webdriver_server,webdriver,script::webdriver_handlers,constellation
```

Tämä yleensä tekee selväksi, käsitelläänkö odotetut API-kutsut.

Kun debuggaat syöte-ongelmia, lisää JS-debug-rivit, jotka loggaavat klikattavan elementin `getBoundingClientRect()`-arvon, ja vertaa compositorin vastaanottamia koordinaatteja manuaalisessa klikkauksessa vs. WebDriver-palvelimen lähettämiä koordinaatteja.
