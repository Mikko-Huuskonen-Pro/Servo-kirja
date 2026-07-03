# Tyyliohje

Suurin osa tyylisuosituksistamme pakotetaan automaattisesti lintereillämme.
Tässä dokumentissa on ohjeita, joita on vaikeampi lintata.

Paljon Servon koodia kirjoitettiin ennen kuin nämä suositukset hyväksyttiin.
Tervetulleita ovat pull requestit, jotka tuovat projektin koodin ajan tasalle nykyaikaisten ohjeiden mukaiseksi.

## Rust

Yleisesti ottaen kaikki Servon Rust-koodi formatoidaan automaattisesti `rustfmt`-työkalulla, kun ajat `./mach fmt`, mikä pitäisi kohdistaa koodisi viralliseen [Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/) -oppaaseen.
Lisäksi Rust-koodin pitäisi noudattaa [Rust API naming conventions](https://rust-lang.github.io/api-guidelines/naming.html) -käytäntöjä.
Nimeämiskäytäntöoppaassa on muutamia epäilmeisiä kohtia, kuten:

- `get_`-etuliitettä ei yleensä käytetä gettereissä Rust-koodissa.
  Poikkeus tähän sääntöön on, kun sitä käytetään `get()`-metodin variantille kuten `std::cell::Cell::get_mut()`.
- Camel case -kirjoituksessa lyhenteet ja yhdys-sanojen supistukset lasketaan yhdeksi sanaksi.
  Esimerkiksi structin pitäisi olla nimeltään `HtmlParser`, ei `HTMLParser`.

### Sisennys

Monimutkaisten funktioiden ja metodien kulun seuraamisen helpottamiseksi pyrimme minimoimaan sisennystä käyttämällä varhaisia return-lauseita.
Tämä vastaa myös monissa tapauksissa spesifikaatioissa käytettyä kieltä.
Kun funktion logiikka saavuttaa 2 tai 3 sisennystasoa, tai kun lyhyt ehtolohko on poikkeustapaus funktion lopussa, harkitse varhaisen return-lauseen käyttöä.
Varhaiset return-lauseet toimivat hyvin yhdistettynä `Option`- tai `enum`-varianttien purkamiseen.
Suosi seuraavaa syntaksia, kun palaat varhaisesti, jos `Option`-arvo on `None`:

```rust
let Some(inner_value) = option_value else {
    return
};
```

Tämä toimii myös `enum`-varianttien kanssa:

```rust
enum Pet {
    Dog(usize),
    Cat(usize),
}

let pet = Pet::Dog(10);
let Pet::Dog(age) = pet else {
   return;
};
```

### Lyhenteet

Servo noudattaa Google C++ -oppaan [nimeämissääntöjä](https://google.github.io/styleguide/cppguide.html#General_Naming_Rules).
Vältä lyhenteitä, joita joku projektin ulkopuolella ei tuntisi.
Älä lyhennä poistamalla kirjaimia sanojen keskeltä.

**Poikkeus:**:
Voit käyttää joitain yleisesti tunnettuja lyhenteitä, kuten `i` silmukkaindeksille.
Voit myös käyttää yksittäisiä kirjaimia, kuten `T` Rust-tyyppiparametreille.

### Enum-variantit

Luettavuuden vuoksi vältä enum-varianttien suoraa `use`-tuontia.
Viittaa sen sijaan enumeihin täydellisellä nimellä (eli `Enum::Variant`).
Nimeäristiriitojen välttämisen lisäksi tämä auttaa koodia tuntemattomia lukemaan helpommin koodissa käytetyn tyypin.

### Kuollut koodi

Lähes kaikissa tapauksissa älä commitoi kuollutta koodia tai kommentoitua koodia repositorioon.
Kommentoitu koodi voi [bit rot](https://en.wikipedia.org/wiki/Software_rot) -ilmiön vuoksi rappeutua helposti, eikä kuollutta koodia testata.
Kun koodista tulee kuollutta, koska sitä ei käytetä lainkaan, se pitäisi poistaa.


**Poikkeus:**
Poikkeus on, kun koodi on kuollutta vain joissakin käännöskonfiguraatioissa.
Siinä tapauksessa käytät `expect(dead_code)` -kääntäjädirektiiviä konfiguraatioqualifierin kanssa.
Esimerkiksi:

```rust
#[cfg_attr(any(target_os = "android", target_env = "ohos"), expect(dead_code))]
pub(crate) const LINE_HEIGHT: f32 = 76.0;
```

Tässä tapauksessa `LINE_HEIGHT`-vakio käännetyään, mutta sen odotetaan olevan kuollutta Android- tai OpenHarmony-käännöksessä.

### `unsafe`-koodi

Pyri välttämään unsafe-koodia.
Valitettavasti selainmoottori on monimutkainen, joten jonkin verran unsafe-koodia on Servossa väistämätöntä.
Jos joudut tekemään `unsafe`-funktion, käytä [Safety comments](https://std-dev-guide.rust-lang.org/policy/safety-comments.html#safety-comments) -kommentteja, jotka selittävät, miksi lohko on turvallinen ja mitkä turvallisuusinvariantit ovat voimassa.
Käytä harkintaa siitä, milloin lisätä safety-kommentteja funktioiden sisällä oleviin `unsafe`-lohkoihin.

### Assertiot

Kun Servon sisäisen logiikan pitäisi tehdä tietystä ehdosta mahdoton, käytä `assert!`- tai `debug_assert!`-lausetta varmistaaksesi sen.
Ajattele `assert!`-lausetta sekä eräänlaisena testinä että dokumentaationa.
Jos assertiossa ilmaistu invariantti joskus muuttuu epätodeksi, Servo voi alkaa kaatua testejä ajaessa, mikä estää logiikkavirheiden pääsyn koodiin.
Lisäksi koodia lukevat voivat tietää vahvemmin kuin kommentti sallii, mitä kirjoittaja oletti todeksi tietyssä kohdassa.
Jos tietty koodin osa on saavuttamaton, esimerkiksi jos enum-variantti on käsitelty aiemmin eikä sitä pitäisi kohdata myöhemmin samassa funktiossa, käytä `unreachable!()`, mutta täytä teksti aina sillä, miksi koodi on saavuttamaton.

### `Option::map` ja `Result::map`/`Result::map_err`

`map`-API:ta pitäisi käyttää vain yhden tyypin muuntamiseen toiseen, ei ohjausvirran muodossa.
Suosi `match`-, `if let`- tai `let`/`else`-rakenteita, kun kirjoitat koodia, joka vaikuttaa vain tiettyyn varianttiin.

### `unwrap()` ja `expect()`

`unwrap`-kutsua `Option`- tai `Result`-tyypille ei pitäisi lähes koskaan käyttää.
Käsittele sen sijaan `None`- tai `Err`-tapaus ja tee tarvittava virheenkäsittely.
Servon ei pitäisi kaatua, kun se on mahdollista.
Jos `None` tai `Err` on mahdoton Servon sisäisen logiikan vuoksi *joka ei liity ulkoiseen syötteeseen tai crateihin*, voit käyttää `expect()`-kutsua kuten assertiota.
`expect()`-kutsulle annetun tekstin pitäisi ilmaista, miksi arvo ei voi olla `None` tai `Err`.

**Poikkeus:**:
Kun käsitellään Rustin `std::sync::Mutex`-tyyppiä tai muita samanaikaisuusprimitiivejä, jotka käyttävät [poisoning](https://doc.rust-lang.org/std/sync/struct.Mutex.html#poisoning)-mekanismia, `unwrap`-kutsu on sopiva.

### `todo!()` ja `unimplemented!()`

Saavutettavassa koodissa älä käytä `todo!`- tai `unimplemented!`-makroja.
Nämä makrot saavat Servon panikoimaan, eikä normaalin verkkosisällön pitäisi saada Servoa panikoimaan.
Sen sijaan pyri tekemään tällaiset tapaukset saavuttamattomiksi ja palauttamaan asianmukaisia virhearvoja tai yksinkertaisesti jättämään koodin tekemättä mitään.

### Makrot

Makrot peittävät toteutustiedot, ja kutsukohdat ovat usein vaikeampia lukea kuin inline Rust-koodi.
Kun mahdollista, suosi geneerisiä/parametrisoituja funktioita `macro_rules!`-makrojen sijaan.
Ilmoitetut makrot pitäisi yrittää näyttää mahdollisimman paljon inline-kelpoiselta Rust-koodilta.

**Poikkeus:**
Kun on yleisiä ohjausvirran kuvioita (esim. virheen tai välimuistissa olevan arvon tarkistus ja varhainen paluu), joita ei voi toistaa `Result`/`Option`-tyypeillä ja `?`-operaattorilla, makrot ovat yksi tapa vähentää toistuvaa boilerplate-koodia.

**Poikkeus:**
Kun monta uniikkia tyyppiä pitää ilmoittaa identtisellä kuviolla, makrot ovat yksi tapa vähentää boilerplate-koodia.

## Shell-skriptit

Shell-skriptit sopivat pieniin tehtäviin tai wrappereihin, mutta on parempi käyttää Pythonia kaikkeen, jossa on vähänkin monimutkaisuutta tai yleisesti ottaen.

Shell-skriptit kirjoitetaan bashilla, alkaen tällä shebangilla:
```
#!/usr/bin/env bash
```

Huomaa, että macOS:n oletusbash on melko vanha, joten ole varovainen uusien ominaisuuksien kanssa.

Skriptien alussa pitäisi ottaa käyttöön muutama valitsin vankkuuden vuoksi:
```
set -o errexit
set -o nounset
set -o pipefail
```

Muista lainata kaikki muuttujat täydellisessä muodossa: `"${SOME_VARIABLE}"`.

Käytä `"$(some-command)"` backtickien sijaan komentokorvauksessa.
Huomaa, että nämäkin pitäisi lainata.

## Servo Book

- Käytä permalinkkejä linkatessasi lähdekoodirepoihin — paina `Y` GitHubissa saadaksesi pysyvän URL-osoitteen

### Markdown-lähde

- Käytä yhtä lausetta per rivi ilman sarakeleveyttä, jotta diffit ja historia ovat helpompia ymmärtää

Lauserivien jakamiseen voit korvata `([.!?]) ` → `$1\n`, mutta varo tapauksia kuten "e.g.".
Yksinkertaisten listojen sisennysten korjaamiseen voit korvata `^([*-] .*\n([* -].*\n)*)([^\n* -])` → `$1  $3`, mutta tämä ei toimi sisäkkäisille tai monimutkaisemmille listoille.

- Johdonmukaisuuden vuoksi sisennä sisäkkäiset listat kahdella välilyönnillä ja käytä `-`-merkkiä järjestämättömissä listoissa

### Notaatio

- Käytä **lihavoitua tekstiä** viitattaessa käyttöliittymäelementteihin kuten valikkovalintoihin, esim. "click **Inspect**"
- Käytä `backtick`-merkkejä viitattaessa yksikirjaimisiin näppäimiin, esim. "press `A` or Ctrl+`A`"

### Virheilmoitukset

- Kun mahdollista, sisällytä aina linkki dokumentaatioon, Zulip-chatiin tai lähdekoodiin — tämä auttaa säilyttämään alkuperäisen kontekstin ja auttaa meitä tarkistamaan ja päivittämään neuvoamme ajan myötä

Loput virheilmoitusten säännöt on suunniteltu varmistamaan, että teksti on mahdollisimman luettavaa ja että lukija voi liittää virheilmoituksensa find-in-page -hakuun mahdollisimman vähällä väärillä osumilla, ilman että säännöt ovat liian hankalia noudattaa.

**Kääri virheilmoitus `<pre><samp>`-elementteihin, `<pre>` rivin alussa (ei sisennettynä).**
Jos haluat tyylitellä virheilmoituksen lainauksena, kääri se `<pre><blockquote><samp>`-elementteihin.

`<pre>` käsittelee rivinvaihdot rivinvaihdoiksi, ja rivin alussa se [estää](https://spec.commonmark.org/0.31.2/#example-169) Markdown-syntaksin vahingossa [aktivoitumisen, kun virheilmoituksessa on tyhjiä rivejä](https://spec.commonmark.org/0.31.2/#example-188).

`<samp>` merkitsee tekstin tietokoneen tuotokseksi, jossa [meillä on CSS](../custom.css), joka saa sen rivittymään kuten terminaalissa.
Koodilohkot (`<pre><code>`) eivät rivity, joten ne voivat tehdä pitkistä virheistä vaikeasti luettavia.

**Korvaa jokainen `&` merkillä `&amp;`, sitten jokainen `<` merkillä `&lt;`.**
Teksti `<pre>`-elementin sisällä ei koskaan käsitellä Markdownina, mutta se on silti HTML-merkkausta, joten se on escapattava.

**Tarkista aina renderöity tulos varmistaaksesi, että kaikki symbolit säilyivät.**
Saatat huomata, että sinun on yhä escapattava osaa Markdownista `\`-merkillä, jotta <samp>called \`Result::unwrap()\` on an \`Err\` value</samp> ei renderöidy muodossa <samp>called `Result::unwrap()` on an `Err` value</samp>.

<table>
<thead>
  <tr>
    <th>Virheilmoitus</th>
    <th>Markdown</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>
<pre><samp>thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: "Could not run `PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=\"1\" PKG_CONFIG_ALLOW_SYSTEM_LIBS=\"1\" \"pkg-config\" \"--libs\" \"--cflags\" \"fontconfig\"`</samp></pre>
    </td>
    <td>
      <pre style="white-space: pre-wrap; word-break: break-all;"><code class="language-html">&lt;pre>&lt;samp>thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: "Could not run `PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=\"1\" PKG_CONFIG_ALLOW_SYSTEM_LIBS=\"1\" \"pkg-config\" \"--libs\" \"--cflags\" \"fontconfig\"`&lt;/samp>&lt;/pre></code></pre>
    </td>
  </tr>
  <tr>
    <td>
<pre><samp>error[E0765]: ...
 --> src/main.rs:2:14
  |
2 |       println!("```);
  |  ______________^
3 | | }
  | |__^</samp></pre>
    </td>
    <td>
      <pre><code class="language-html">&lt;pre>&lt;samp>error[E0765]: ...
 --> src/main.rs:2:14
  |
2 |       println!("```);
  |  ______________^
3 | | }
  | |__^&lt;/samp>&lt;/pre></code></pre>
    </td>
  </tr>
</tbody>
</table>
