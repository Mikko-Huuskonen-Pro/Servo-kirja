# Crate-riippuvuudet

Rust-kirjastoa kutsutaan crateksi.
Servo käyttää paljon crateja.
Nämä cratet ovat riippuvuuksia.
Ne on listattu tiedostoissa nimeltä `Cargo.toml`.
Servo on jaettu komponentteihin ja portteihin (katso `components`- ja `ports`-hakemistot).
Jokaisella on omat riippuvuutensa ja oma `Cargo.toml`-tiedostonsa.

`Cargo.toml`-tiedostot listaavat riippuvuudet.
Voit muokata tätä tiedostoa.

Esimerkiksi `components/net_traits/Cargo.toml` sisältää:

```
 [dependencies.stb_image]
 git = "https://github.com/servo/rust-stb-image"
```

Mutta koska `rust-stb-image`-API voi muuttua ajan myötä, ei ole turvallista kääntää `rust-stb-image`-repon `HEAD`-versiota vastaan.
`Cargo.lock`-tiedosto on `Cargo.toml`-tiedoston tilannekuva, joka sisältää viitteen tarkkaan revisioon ja varmistaa, että kaikki kääntävät aina samalla konfiguraatiolla:

```
[[package]]
name = "stb_image"
source = "git+https://github.com/servo/rust-stb-image#f4c5380cd586bfe16326e05e2518aa044397894b"
```

Tätä tiedostoa ei pidä muokata käsin.
Normaalissa Rust-projektissa git-revisiota päivittäisi komennolla `cargo update -p stb_image`, mutta Servossa käytä `./mach cargo-update -p stb_image`.
Myös muut cargo-argumentit ymmärretään, esim. käytä --precise '0.2.3' päivittääksesi kyseisen craten versioon 0.2.3.

Katso [Cargon dokumentaatio Cargo.toml- ja Cargo.lock-tiedostoista](https://doc.rust-lang.org/cargo/guide/cargo-toml-vs-cargo-lock.html).

# Työskentely craten parissa

Kuten yllä selitetään, Servo riippuu monista kirjastoista, mikä tekee siitä hyvin modulaarisen.
Työskennellessäsi Servon bugiin päädyt usein johonkin sen riippuvuuksista.
Haluat silloin kääntää oman versionsi riippuvuudesta (ja ehkä kääntäminen kirjaston `HEAD`-versiota vastaan korjaa ongelman!).

Esimerkiksi yritän tuoda joitain cocoa-tapahtumia Servoon.
Servon ikkuna työpöydällä rakennetaan kirjastolla nimeltä [winit](https://github.com/rust-windowing/winit).
winit riippuu cocoa-kirjastosta nimeltä [cocoa-rs](https://github.com/servo/cocoa-rs).
Kun rakennat Servon, kaikki nämä riippuvuudet ladataan ja käännätään automaattisesti.
Mutta koska haluan työskennellä tämän cocoa-tapahtumaominaisuuden parissa, haluan Servon käyttävän omaa versiotani _winit_- ja _cocoa-rs_-kirjastoista.

Projektini on järjestetty näin:

```
~/my-projects/servo/
~/my-projects/cocoa-rs/
```

Molemmat kansiot ovat git-repositorioita.

Jotta Servo käyttäisi polkua `~/my-projects/cocoa-rs/`, selvitä ensin, mitä versiota cratesta Servo käyttää ja onko se git- vai crates.io-riippuvuus.

Molemmat tiedot löytyvät komennolla, tässä esimerkissä, `cargo pkgid cocoa` (`cocoa` on paketin nimi, joka ei välttämättä vastaa repo-kansion nimeä).

Jos tulos on muodossa `https://github.com/servo/cocoa-rs#cocoa:0.0.0`, kyseessä on git-riippuvuus ja sinun on muokattava tiedostoa `~/my-projects/servo/Cargo.toml` ja lisättävä loppuun:

```toml
[patch]
"https://github.com/servo/cocoa-rs#cocoa:0.0.0" = { path = '../cocoa-rs' }
```

Jos tulos on muodossa `https://github.com/rust-lang/crates.io-index#cocoa#0.0.0`, kyseessä on crates.io-riippuvuus ja sinun on muokattava tiedostoa `~/my-projects/servo/Cargo.toml` seuraavasti:

```toml
[patch]
"cocoa:0.0.0" = { path = '../cocoa-rs' }
```

Molemmat kertovat mille tahansa cargo-projektille, ettei se käytä riippuvuuden verkkoversiota vaan paikallista klooniasi.

Lisätietoja riippuvuuksien ohittamisesta: [Cargon dokumentaatio](https://doc.crates.io/specifying-dependencies.html#overriding-dependencies).

# Crate-julkaisujen pyytäminen

Servon selainmoottorin luomisen lisäksi Servo-projekti julkaisee modulaarisia komponentteja, kun ne voivat hyödyttää laajempaa Rust-kehittäjäyhteisöä.
Esimerkki tällaisesta cratesta on [`rust-url`](https://crates.io/crates/url).
Pyrimme olemaan hyviä ylläpitäjiä, mutta selainmoottorin ja ulkoisten kirjastojen kokoelman hallinta voi olla paljon työtä, joten emme takaa säännöllisiä julkaisuja näille modulaarisille crateille.

Jos koet, että jonkin näistä crateista julkaisu on ajankohtainen, vastaamme pyyntöihin uusista julkaisuista.
Uuden julkaisun pyytämisen prosessi on:

1. Luo yksi tai useampi pull request, joka valmistelee craten uutta julkaisua varten.
2. Luo pull request, joka nostaa versionumeron repositoriossa, ollen tarkkana siitä, mikä version osa pitäisi kasvaa edellisestä julkaisusta. Tämä tarkoittaa, että sinun on ehkä merkittävä, onko mukana breaking change -muutoksia.
3. Pyydä pull requestissa uuden version julkaisemista. Muutoksen landaava henkilö on vastuussa uuden version julkaisemisesta tai selittää, miksi sitä ei voida julkaista pull requestin landauksen yhteydessä.
