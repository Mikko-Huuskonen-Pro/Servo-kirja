# DOM Bindings

DOM-bindings ovat [WebIDL](https://en.wikipedia.org/wiki/Web_IDL) -rajapintojen toteutuksia natiivissa Rust-koodissa.
Koodigeneraattori tuottaa liimakoodia, joka altistaa nämä natiivi Rust -toteutukset JavaScriptille SpiderMonkey API:n kautta.
WebIDL-tiedostot sijaitsevat `components/script_bindings/webidls/` -hakemistossa.
Nämä tiedostot sisältävät kunkin rajapinnan määrittelyn, mukaan lukien niiden nimet, attribuutit ja metodit.
Näiden rajapintojen Rust-toteutukset sijaitsevat `components/script/dom` -hakemistossa.
Jokainen Rust-toteutus on erityinen Rust `struct`, joka sisältää kunkin DOM-objektin tilan.


## Layout Wrappers

JavaScript ajaa yhdellä säieellä, eikä DOM-rajapinnat ole säie-turvallisia.
Layoutin täytyy päästä käsiksi DOM:iin, mutta sen odotetaan ajavan eri säieillä.
Teoriassa tämä voi toimia, kunhan säie ei yritä lukea tai mutatoida DOM-objektia, kun toinen säie mutatoi samaa objektia.
Yritämme tehdä tästä unsafety-osiosta helpommin hallittavan rajoittamalla, miten layout voi käyttää DOM-objekteja wrapper `struct`:in kautta, joka altistaa rajoitetun, mutta yhteensopivan, toiminnallisuuden.

On kaksi käyttömallia:

1. Layout olettaa itse, että vain yksi säie "omistaa" DOM-wrapperin jokaiselle solmulle, joten DOM:iin kirjoittaminen pitäisi olla turvallista.
   Solmun lapset voidaan käsitellä muilla säieillä samanaikaisesti.
   Toisaalta vanhempaan pääsy on erittäin vaarallista, koska toinen säie voi kirjoittaa ja lukea vanhempasolmua.
2. `stylo` ja `selectors` olettavat, että solmuihin pääsee mistä tahansa säieestä, mutta vain yksi säie kirjoittaa solmuun kerrallaan.
   Tämä tarkoittaa, että vanhempiin ja lapsiin pääsy on turvallista, mutta solmuun kirjoittaminen on erittäin vaarallista.

Lisäksi `layout` riippuu `script`:ista, joten DOM-instanssien välittäminen suoraan `script`:ista `layout`:iin ei ole mahdollista, muuten meillä olisi riippuvuussykli.
Sen sijaan `layout-api` altistaa trait-pohjaisen rajapinnan ja `script` toteuttaa sen.
Tämä mahdollistaa `Layout`-rajapinnan käsittelemään solmuja suoraan.

Neljä trait:ia, joita altistamme, ovat:

 - `LayoutNode`: Tämä on perusrajapinta DOM-solmulle, jota käytetään layoutissa.
 - `DangerousStyleNode`: Tämä on rajapinta, joka toteuttaa `stylo`- ja `selectors`-trait:t solmujen kanssa vuorovaikutukseen.
    Tämä voidaan luoda `LayoutNode`:sta kutsumalla unsafe-metodia `LayoutNode::dangerous_style_node()`.
    Yleensä näitä solmuja ei tulisi käyttää layout-koodissa, ellei niitä välitetä suoraan `stylo`- tai `selectors`-kutsuun.
 - `LayoutElement`: Tämä on perusrajapinta DOM-elementille, jota käytetään layoutissa.
 - `DangerousStyleElement`: Tämä on rajapinta, joka toteuttaa `stylo`- ja `selectors`-trait:t elementtien kanssa vuorovaikutukseen.
    Tämä voidaan luoda `LayoutElement`:ista kutsumalla unsafe-metodia `LayoutElement::dangerous_style_element()`.
    Yleensä näitä elementtejä ei tulisi käyttää layout-koodissa, ellei niitä välitetä suoraan `stylo`- tai `selectors`-kutsuun.

`script` toteuttaa nämä trait:t tyypeillä `ServoLayoutNode`, `ServoDangerousStyleNode`, `ServoLayoutElement` ja `ServoDangerousLayoutElement`.
Lisäksi `script` altistaa kaksi muuta struct:ia, jotka toteuttavat `stylo`-trait:t: `ServoDangerousStyleDocument` ja `ServoDangerousStyleShadowRoot`.

### Säännöt Layout Wrappers -käytölle

 - Yksinkertaisuuden ja nopeampien käännösaikojen vuoksi trait-määrittelyissä (`LayoutNode` ja `LayoutElement`) ei tulisi olla oletusmetodeja.
   Kaikki toteutuskoodi tulisi olla `script`:issa.
 - Layout *ei saisi* käyttää `DangerousStyleNode`:a ja `DangerousStyleElement`:iä, ellei se kutsu `stylo`:on tai `selectors`:iin.
   Tällä hetkellä on muutamia poikkeuksia, mutta ne poistetaan vähitellen.
 - Layout ei saisi luottaa metodeihin, jotka on määritelty vain `ServoLayoutNode`:ssa ja `ServoLayoutElement`:issa.
   Sen sijaan uutta toiminnallisuutta tulisi lisätä `LayoutNode`- tai `LayoutElement`-trait:eihin ja sitten toteuttaa `ServoLayoutNode`:ssa tai `ServoLayoutElement`:issa.
   Tämä mahdollistaa `TrustedNodeAddress`:in poistamisen tulevaisuudessa ja `LayoutNode`:ien välittämisen suoraan layoutille, poistaen unsafe-koodin lähteen.
