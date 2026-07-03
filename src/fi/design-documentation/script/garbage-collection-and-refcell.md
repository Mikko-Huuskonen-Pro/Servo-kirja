# Roskienkeruu ja RefCell

Servon integraatio SpiderMonkeyn roskienkeruujärjestelmään ja Rustin malli jaetulle omistajuudelle vuorovaikuttavat hienovaraisesti.
Koska DOM-objektit Servossa eivät ole uniikisti omistettuja, meidän täytyy käyttää `RefCell`/`DomRefCell`:iä jäsenille, joita voidaan mutatoida.

Kun roskienkeruu (GC) käynnistetään SpiderMonkeysta, jokainen DOM-objekti jäljitetään löytääkseen saavutettavat JS-arvot.
Tämä jäljitys on toteutettu `JSTraceable`-derive:llä, joka kutsuu `JSTraceable::trace`:a jokaiselle DOM-objektin jäsenelle (ellei sitä ole annotoitu `#[no_trace]`-attribuutilla).

Koska `RefCell`:in `JSTraceable`-toteutus lainaa solua, tämä tarkoittaa, että mikä tahansa DOM-objektin jäsenen mutable-laina aiheuttaa paniikin, jos GC tapahtuu, kun laina on vielä aktiivinen.
Kutsumme tätä usein Servossa **borrow hazard** -tilanteeksi.

## Borrow hazard -tilanteiden tunnistaminen: `CanGc`

Servossa on tyyppi nimeltä [`CanGc`](https://github.com/servo/servo/blob/9fa6303d261f2aca3a19448fefda69280c4d8892/components/script_bindings/script_runtime.rs#L48-L62), jota käytetään osoittamaan, milloin GC voi tapahtua ennen kuin kutsuttu funktio palaa.
On yksi sääntö: kun kutsutaan funktiota, joka hyväksyy `CanGc`-argumentin, kutsujan täytyy myös hyväksyä `CanGc`-argumentti.

Tähän sääntöön on poikkeuksia:
* trait-metodit, jotka on määritelty `script`/`script_bindings`-cratejen ulkopuolella, eivät voi propagoida `CanGc`:ä, joten toteutusten täytyy käyttää `CanGc::note()`:a, jos kutsuttu funktio vaatii `CanGc`-argumentin 
* asynkronisten tehtävien täytyy käyttää `CanGc::note()`:a, koska ne suoritetaan riippumatta kutsujan stack frame:sta
* `extern "C"`-funktioiden täytyy käyttää `CanGc::note()`:a, koska ne vaativat vastaavan allekirjoituksen ulkoiselle kirjastolle

Kun `CanGc` propagoidaan oikein koodin läpi, borrow hazard -tilanteet voidaan tunnistaa etsimällä `borrow_mut()`-käyttöjä `can_gc`:n käytön läheltä.
Erityisesti, kun `borrow_mut()`:n palautusarvo tallennetaan muuttujaan, ja muuttuja on vielä elossa, kun funktiokutsu sisältää `can_gc`-argumentin, on erittäin todennäköistä, että kyseessä on paniikki odottamassa laukeamistaan!

Katso [esimerkki-issue](https://github.com/servo/servo/issues/39947), joka korostaa borrow hazard -tilannetta.
Lue lisää [alkuperäisestä issue:sta](https://github.com/servo/servo/issues/33140), joka ehdotti staattista analyysiä.

## Borrow hazard -tilanteiden varmentaminen

Varmistaaksemme, että tietty mutable-laina voi laukaista paniikin GC:n tapahtuessa, tarvitsemme 1) deterministisen roskienkeruun, 2) tavan ajaa epäilyttävä koodi.

Tehdäksesi roskienkeruusta deterministisen, sinun täytyy ensin kääntää Servo `--debug-mozjs`:llä, sitten ajaa se `--pref js_mem_gc_zeal_level=2 --pref js_mem_gc_zeal_frequency=1` -asetuksilla.
Tämä ottaa käyttöön tilan, jossa roskienkeruu ajetaan aina JS-allokaation tapahtuessa, ja se varmasti laukaisee kaikki piilevät borrow hazard -tilanteet.
Se on myös erittäin hidas, joten testitapauksen minimointi säästää aikaa.

Jos et ole varma, miten epäilyttävä koodi laukaistaan, lisää siihen paniikki ja aja WPT-testejä sopivasta hakemistosta, kunnes löydät testitiedoston, joka panikoi.

## Mallit borrow hazard -tilanteiden korjaamiseen

* Pakota laina pudotettavaksi aiemmin scopettamalla se (`{ ... }`)
* Kloonaa väliaikainen arvo lainatusta arvosta, jotta laina voidaan pudottaa aiemmin
* Sen sijaan, että käytät `RefCell<SomeStruct>`:ia, tee `SomeStruct`:in jäsenet käyttämään `RefCell`/`Cell`:iä
* Jaa sekoitettu immutable/mutable-laina useiksi scopatuiksi immutable-lainauksiksi ja käytä mutable-lainauksia vain mutaation tapahtuessa

## Esimerkkejä borrow hazard -tilanteiden korjaamisesta

* https://github.com/servo/servo/pull/40139
* https://github.com/servo/servo/pull/40138

## Esimerkkejä CanGc-argumenttien propagoinnista

* https://github.com/servo/servo/pull/40033
* https://github.com/servo/servo/pull/36180
* https://github.com/servo/servo/pull/40325

## `CanGc`-argumenttien lisääminen generoituihin DOM-metoditrait:eihin

WebIDL-metodien ylimääräiset argumentit kontrolloidaan [`Bindings.conf`](https://github.com/servo/servo/blob/main/components/script_bindings/codegen/Bindings.conf) -tiedostolla.
`CanGc`-argumentit kontrolloidaan erityisesti `canGc`-avaimella tietylle rajapinnalle.
Jos rajapintaa ei ole vielä listattu tiedostossa, voit vapaasti lisätä sen.
