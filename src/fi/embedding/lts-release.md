# Servon LTS-julkaisut

Kuukausittaisten julkaisujemme lisäksi Servo-yhteisö tarjoaa parhaan ponnistelun mukaisia pitkäaikaistuen (LTS) -julkaisuja.
Koska Servon julkinen API on vielä kehittymässä, nämä LTS-julkaisut voivat sopia upottajille, joilla on rajalliset resurssit pysyä ajan tasalla uusimmasta Servo-julkaisusta.
LTS-julkaisut antavat upottajille aikataulutetut päivitysikkunat ja hyödyn tietoturvakorjauksista, mukaan lukien JavaScript-moottorimme tietoturvakorjaukset.

## LTS-julkaisujen laajuus

**Servo toimitetaan SELLAISENAAN eikä sille anneta erityisiä takuita.**
LTS-julkaisut (ja alla olevat yksityiskohdat) tarjotaan parhaan ponnistelun mukaisesti.
Toistaiseksi tämä tarkoittaa:

- Uusi LTS-julkaisu / haara otetaan käyttöön 6 kuukauden välein, perustuen silloisen tavallisen julkaisun versioon.
- Odotettu tukikesto on 9 kuukautta, mikä antaa upottajille aikaa siirtyä seuraavaan LTS-julkaisuun.
- LTS-julkaisu saa vain tietoturvakorjauksia.
- Korjausjulkaisut tehdään tarpeen mukaan; kiinteää aikataulua ei ole.
- Laajuudessa on vain `servo`-kirjasto ja sen riippuvuudet.
  Selaindemo servoshell on nimenomaisesti laajuuden ulkopuolella.
- Tuettua vähimmäis-Rust-versiota (MSRV) ei nosteta LTS-haarassa, mutta se voi nousta siirryttäessä seuraavaan LTS-versioon
- Julkaisut julkaistaan crates.io:hon **jos mahdollista**, mutta upottajien tulee varautua siihen, että `git`-riippuvuuksia saatetaan vaatia.

## CVE-korjausten paikkaaminen alavirta-cratesissa

Monet Rust-kirjastot eivät yleensä tuo CVE-korjauksia vanhempiin julkaisuihin / haaroihin.
Koska MSRV-nostot käsitellään taaksepäin yhteensopimattomina muutoksina, tämä voi johtaa siihen, ettei voimme päivittää kirjaston uudempaan julkaistuun versioon (joka korjaa CVE:n).
Nämä tilanteet käsitellään tapauskohtaisesti, ihanteellisesti yhteistyössä upstream-ylläpitäjän kanssa, ja ne sisältävät todennäköisesti kirjaston paikatun version hakemisen `git`:n kautta.
Tämä tarkoittaa, että Servon LTS-korjausjulkaisuja ei pidä odottaa crates.io:ssa, koska niissä voi olla `git`-riippuvuuksia.

## Rajoitukset

- Servo toimitetaan SELLAISENAAN eikä sille anneta erityisiä takuita, mukaan lukien tietoturvatakuut.
  LTS-julkaisut tarjotaan parhaan ponnistelun mukaisesti kiinnostuneiden yhteisön jäsenten toimesta.
- Kuten yllä mainittiin, Servolla ei ole vielä 1.0-julkaisua, mikä tarkoittaa että Servon tuotantokäyttöä tulee arvioida huolellisesti.
  Servon käyttö sovelluksessa tunnetun, luotetun sisällön renderöintiin on hyvin erilainen riskiprofiili kuin Servon käyttö selaimena mielivaltaisen sisällön renderöintiin.

## LTS-julkaisujen ylläpitäjät

- @jschwe (Jonathan Schwender)
- TBD
