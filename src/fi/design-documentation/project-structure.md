<!-- TODO: needs copyediting -->

# Projektin rakenne

- **components**
  - **bluetooth** — Bluetooth-säieen toteutus.
  - **canvas** — 2D- ja WebGL-canvasien maalaussäieiden toteutus.
  - **compositing** — Integraatio OS:n ikkunointi/renderöinti- ja tapahtumasilmukkaan.
  - **constellation** — Resurssien hallinta ylätason selauskontekstille (ts. välilehdelle).
  - **devtools** — Prosessin sisäinen palvelin selaininstanssien manipulointiin etä-Firefox-kehittäjätyökaluasiakkaan kautta.
  - **fonts** — Koodi fonttien ja tekstin muotoilun käsittelyyn.
  - **layout** — Muuntaa sivun sisällön sijoitetuiksi, tyylitetyiksi laatikoiksi ja välittää tuloksen rendererille.
  - **layout_thread** — Ajaa layout-säieitä, viestii script-säieen kanssa ja kutsuu layout-cratea layoutin suorittamiseksi.
  - **msg** — Jaetut API:t viestintään tiettyjen säieiden ja cratejen välillä.
  - **net** — Verkkoprotokollien toteutukset sekä tilan ja resurssien hallinta (välimuisti, evästeet jne.).
  - **plugins** — Syntaksilaajennukset, mukautetut attribuutit ja lintit.
  - **profile** — Muistin ja ajan profilointityökalut.
  - **script** — DOM:n toteutus (natiivi Rust-koodi ja bindings SpiderMonkeyhin).
  - **script_bindings** - Tukikoodi ja WebIDL-tiedostoista generoidut bindings.
    Bindings koostuvat trait:eista, jotka edustavat WebIDL-rajapintoja, ja liimakoodista
  SpiderMonkey JavaScript -moottorille.
    Varsinaiset trait-toteutukset sijaitsevat `script`-crate:ssa.
    Nämä on jaettu kahteen crateen inkrementaalisten käännösten nopeuttamiseksi.
  - **script_layout_interface** — API, jonka script-crate tarjoaa layout-cratelle.
  - **selectors** — CSS-selektorien täsmäytys.
  - **servo** — Entry pointit servo-sovellukselle ja libservo-upotuskirjastolle.
  - **shared** — Jaetut trait:t/koodi, joita useat komponentit käyttävät ilman riippuvuutta pääcrateen käännösnopeussyistä.
  - **style** — API:t CSS:n jäsentämiseen ja tyylitiedostojen ja tyylitettyjen elementtien käsittelyyn.
  - **util** — Sekalaisia apumetodeja ja -tyyppejä, joita käytetään laajasti projektissa.
  - **webdriver_server** — Prosessin sisäinen palvelin selaininstanssien manipulointiin WebDriver-asiakkaan kautta.
  - **webgpu** — WebGPU API:n säieiden toteutus.
- **etc** — Hyödyllisiä työkaluja ja skriptejä kehittäjille.
- **ports**
  - **servoshell** — Esimerkkiselain, joka käyttää servoa.
- **python**
  - **mach** — Komentorivityökalu kehittäjätehtävien helpottamiseen.
  - **servo** — Servo-spesifisten mach-komentojen toteutukset.
  - **tidy** — Python-paketti koodilinteistä, jotka ajetaan automaattisesti ennen muutosten yhdistämistä.
- **resources** — Ajonaikaiset tiedostot.
  Ne täytyy jotenkin sisällyttää, kun binäärikäännöksiä jaetaan.
- **support**
  - **android** — Kirjastot, jotka vaativat erityiskäsittelyä Android-alustoille kääntäessä
- **target**
  - **debug** — `./mach build --debug` -komennolla generoidut käännösartefaktit.
  - **doc** — Dokumentaatio generoidaan tänne `rustdoc`-työkalulla ajettaessa `./mach doc`
  - **release** — `./mach build --release` -komennolla generoidut käännösartefaktit.
- **tests**
  - **dromaeo** — Harness Dromaeo-testisarjan automaattiseen ajoon.
  - **html** — Manuaaliset testit ja kokeilut.
  - **jquery** — Harness jQuery-testisarjan automaattiseen ajoon.
  - **power** — Työkalut virrankulutuksen mittaamiseen.
  - **unit** — Yksikkötestit rustc:n sisäänrakennetulla test harnessilla.
  - **wpt** — W3C web-platform-tests ja csswg-tests työkaluineen niiden ajamiseen ja odotettuihin epäonnistumisiin.

# Repositoriot

Servo-projekti ylläpitää useita repositorioita, jotka joko julkaistaan Servosta riippumatta tai on forkattu upstream-projektista.

## Laajasti käytetty Rust-ekosysteemissä

- [euclid](https://github.com/servo/euclid): Geometriset tyypit
- [ipc-channel](https://github.com/servo/ipc-channel): Prosessienvälinen viestintäkanava
- [html5ever](https://github.com/servo/html5ever): HTML5-parseri Rustilla
- [rust-cssparser](https://github.com/servo/rust-cssparser): CSS-parseri Rustilla
- [rust-url](https://github.com/servo/rust-url): URL-kirjasto Rustille, perustuu [URL Standardiin](https://url.spec.whatwg.org/). Tunnetaan myös nimellä `url`.
- [string-cache](https://github.com/servo/string-cache): Merkkijonojen internointikirjasto

## Forkit

- [mozjs](https://github.com/servo/mozjs): Servon fork SpiderMonkeysta ja Rust-bindingsista
- [stylo](https://github.com/servo/stylo): Servon CSS-toteutus säännöllisellä synkronoinnilla upstream-versioon Gecko-repositoriossa
- [webrender](https://github.com/servo/webrender): Firefoxin WebRenderin fork pienillä Servo-spesifisillä muutoksilla

## Servo-sisäiset

- [book](https://github.com/servo/book): Tämä kirja!
- [ci-runners](https://github.com/servo/ci-runners): Skriptit ja työkalut Servon CI:lle (continuous integration)
- [malloc_size_of](https://github.com/servo/malloc_size_of): Arvojen ajonaikaisen koon mittaus
- [media](https://github.com/servo/media): Mediat backend, jota Servo käyttää, tällä hetkellä vain GStreamer
- [servo](https://github.com/servo/servo): Päärepositorio Servo web platform -moottorille
- [surfman](https://github.com/servo/surfman): Matalan tason cross-platform Rust-kirjasto graafisten pintojen hallintaan
- [wpt](https://github.com/servo/wpt): Servon fork Web Platform Testsistä
