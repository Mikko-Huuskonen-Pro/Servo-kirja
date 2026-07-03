## Uuden Servo-version julkaiseminen

### Servo-version nostaminen main-haarassa

Tällä hetkellä julkaisemme uuden Servo-version kuukausittain.
Kuukauden lopussa valmistele haara viimeisimmän `main`-haaran pohjalta.
Aja `./mach release X.Y.Z` nostaaksesi versionumerot ja commitoi muutokset.
Avaa pull request servossa yhdistääksesi haaran `main`-haaraan.
Ihannetapauksessa pull request yhdistetään ajoissa kuukauden viimeisenä tai seuraavan kuukauden ensimmäisenä päivänä, jotta versionumeron nosto korreloi läheisesti blogipostin aikavälin kanssa.
Blogipostin aikaväli on viimeisen kuukauden yöjulkaisun päähän asti (mukaan lukien).
Versionoston pitäisi yhdistyä **sen jälkeen**, jotta kaikki blogipostissa mainitut muutokset sisältyvät julkaisuun.

### Julkaisuhaaran luominen

Luo haara nimellä `release/vX.Y.Z` versionumeron nostaneen commitin pohjalta (`main`-haarassa).
Jos yhdistämispäivä oli (merkittävästi) myöhemmin ja merkittäviä muutoksia tehtiin, julkaisuhaara voidaan perustaa myös aiempaan committiin ja versionumeron nosto backportata / tehdä uudelleen.
Haara pitää pushata upstream servo -repositorioon.

### Luonnosjulkaisun luominen testausta varten

Siirry servo-repositorion `actions`-välilehteen ja valitse [`Release`-workflow](https://github.com/servo/servo/actions/workflows/release.yml).
Valitse `Run workflow` -painike oikeasta yläkulmasta.
Valitse haara `release/vX.Y.Z` (jonka juuri pushasit) workflow ajettavaksi.
Jätä valintaruutu **valitsematta**, jotta julkaisu luodaan **nightly-releases-repositorioon**, koska se mahdollistaa ei-maintainerien auttamisen julkaisun testauksessa.
Anna `tag`-arvoksi `vX.Y.Z-beta1` luodaksesi esijulkaisun testausta varten.
Klikkaa `Run workflow`.
Luotu workflow ajaa ja luo (julkisen) julkaisun [nightly-releases-repositorioon](https://github.com/servo/servo-nightly-builds/releases).
Kun julkaisu on luotu, voit avata ketjun Zulipissa ja tehdä kutsun eri artefaktien testaamiseen.

### Muutosten backporttaaminen julkaisuhaaraan

Jos julkaisuhaaraan täytyy soveltaa kriittisiä korjauksia (esim. manuaalisessa testauksessa havaittujen kaatumisten korjaus), PR pitää avata haaraa vastaan tavanomaisen review-prosessin mukaisesti.
Tämän jälkeen pitäisi suorittaa uusi testauskierros, joten backporttaa vain välttämättömissä tapauksissa.

### Julkaisun valmistelu

Kun testijulkaisu nightly-repositoriossa on testattu manuaalisesti, voit valmistella julkaisun ajamalla release-workflow uudelleen, mutta tällä kertaa valitsemalla ruudun luodaksesi julkaisun upstream servo -repositorioon.
Tämä luo vain luonnosjulkaisun upstream servo -repositorioon.
Kun artefaktit on ladattu julkaisuun, klikkaa muokkauspainiketta muokataksesi luonnosjulkaisua.
Tagin pitäisi jo oikein olla `vX.Y.Z`.
Muuta `target` arvosta `main` arvoon `release/vX.Y.Z`.
Klikkaa `Generate release notes` ja kääri sitten generoidut release notes seuraavaan lohkoon:

```
<details>
  <summary>Generated Release notes</summary>
  Release notes here.
</details>
```

Ota yhteyttä macOS-artefaktin allekirjoittajaan ja pyydä häntä allekirjoittamaan julkaisu.
Tämä voi kestää jonkin aikaa, joten se pitäisi tehdä pari päivää ennen suunniteltua julkaisupäivää.

Lopuksi lisää tavanomainen release notes -yhteenvetomme, jossa linkki blogipostiin ja yleisiin ongelmiin (katso aiempien release notes -kohtien esimerkit).
Kun blogiposti on julkaistu, julkaisemme julkaisun.
