# Arkkitehtuuri

Servo on projekti uuden verkkoselaimen moottorin kehittämiseksi.
Tavoitteemme on luoda arkkitehtuuri, joka hyödyntää rinnakkaisuutta monilla tasoilla samalla kun eliminoimme yleisiä virheiden ja tietoturva-aukkojen lähteitä, jotka liittyvät virheelliseen muistinhallintaan ja datakilpailuihin.

Koska C++ sopii huonosti näiden ongelmien estämiseen, Servo on kirjoitettu [Rustilla](http://rust-lang.org), modernilla kielellä, joka on suunniteltu erityisesti Servon vaatimusten huomioon ottaen.
Rust tarjoaa tehtäväpohjaisen rinnakkaisuusinfrastruktuurin ja vahvan tyyppijärjestelmän, joka pakottaa muistiturvallisuuden ja datan kilpailuttamattomuuden.

```mermaid
flowchart TB
    subgraph Web-sisällön prosessi
    ScriptA[Script-säie]-->PipelineA[Pipeline A]
    PipelineA
    PipelineA-->ImageA[Kuvavälimuisti]
    PipelineA-->FontA[Fonttivälimuisti]
    PipelineA-->LayoutA[Layout]
    LayoutA-->ImageA
    LayoutA-->FontA

    ScriptA[Script-säie]-->PipelineB[Pipeline B]
    PipelineB
    PipelineB-->ImageB[Kuvavälimuisti]
    PipelineB-->FontB[Fonttivälimuisti]
    PipelineB-->LayoutB[Layout]
    LayoutB-->ImageB
    LayoutB-->FontB
    end

    subgraph Upotusprosessi
    direction TB
    Embedder-->Constellation[Constellation-säie]
    Embedder<-->Renderer[Renderer]
    Constellation<-->Renderer
    Renderer-->WebRender
    Constellation-->SystemFont[Järjestelmäfonttivälimuisti]
    Constellation-->Resource[Resurssienhallinta]
    end

    Constellation<-->ScriptA
    FontA-->SystemFont
    PipelineA-->Renderer
    FontB-->SystemFont
    PipelineB-->Renderer
 ```

Tämä kaavio näyttää moniprosessisen Servon arkkitehtuurin, kun se ajetaan yhdellä web-sisällön prosessilla.
Kun moniprosessitila on käytössä, jokainen script-säie ajaa omassa web-sisällön prosessissaan.
Kun moniprosessitila on pois päältä, kaikki script-säieet ajavat upotusprosessissa.
Jokaisella script-säieellä on joukko viestintäkanavia constellationin ja upotusprosessin upotus-API-osien kanssa.
Yhtenäiset viivat osoittavat viestintäkanavia tai API-kutsuja.

## Constellation, script-säieet ja pipeline:t

Jokaisella Servo-instanssilla on yksi [constellation](https://github.com/servo/servo/blob/main/components/constellation/lib.rs), joka hallinnoi web-sisällön prosesseja kaikille kehyksille kaikissa `WebView`-instansseissa.
Web-sisällön prosessin script-säie voi hallita useita pipelineja, yhden jokaiselle `<iframe>`-elementille tai `WebView`-instanssin pääkehykselle.
Script-säieen pipeline vastaa syötteen vastaanottamisesta, JavaScriptin suorittamisesta DOM:ia vasten, layoutin suorittamisesta, display listien rakentamisesta ja display listien lähettämisestä rendererille.
Servo-instanssilla on yksi renderer koko instanssille, ja se hallinnoi useita WebRender-instansseja, jotka renderöivät erilaisiin `RenderingContext`-instansseihin (käytännössä OpenGL-konteksteihin alustan pinnoilla).

Pipeline koostuu kolmesta pääosasta:

* **[Script](script.md)**: Scriptin ensisijainen tehtävä on luoda ja omistaa DOM ja suorittaa JavaScript-moottoria.
  Se vastaanottaa tapahtumia useista lähteistä, mukaan lukien navigointitapahtumat, ja reitittää ne tarpeen mukaan.
* **[Layout](layout.md)**: Layout käynnistyy aluksi samalla säieellä kuin Script, mutta se voi käyttää worker-säieitä sivun layoutin rinnakkaiseen suorittamiseen. Se laskee tyylit ja rakentaa kaksi päälayout-tietorakennetta, *[box tree](layout.md#box-tree)* ja *[fragment tree](layout.md#fragment-tree)*.
  Fragment treeä käytetään solmujen muuntamattomien sijaintien määrittämiseen ja siitä rakennetaan display list, joka lähetetään rendererille.
* **[Renderer](compositor.md)**: Renderer (tunnetaan myös compositorina) välittää display listit [WebRenderille](https://github.com/servo/webrender), joka on sisällön rasterointi- ja näyttömoottori, jota sekä Servo että Firefox käyttävät.
  Se käyttää GPU:ta sivun lopullisen kuvan renderöintiin.
  Upottajan käyttöliittymäsäieellä ajettava renderer vastaanottaa myös ensimmäisenä syöte-tapahtumat, jotka yleensä lähetetään heti constellationille ja sitten script-säieelle käsittelyä varten.
  Jotkin tapahtumat, kuten scroll- ja kosketustapahtumat, voidaan käsitellä aluksi rendererissä responsiivisuuden vuoksi.

## Rinnakkaisuus ja rinnakkaisuus (concurrency vs parallelism)

Rinnakkaisuus (concurrency) on tehtävien erottelua vuorottelevan suorituksen tarjoamiseksi.
Rinnakkaisuus (parallelism) on useiden työpalojen samanaikaista suorittamista nopeuden lisäämiseksi.
Näin hyödynnämme molempia:

* **Tehtäväpohjainen arkkitehtuuri**:
  Järjestelmän pääkomponentit tulisi jakaa näyttelijöiksi (actors) eristetyillä keoilla, selkeillä raja-arvoilla virheille ja palautumiselle.
  Tämä kannustaa myös löyhään kytkentään koko järjestelmässä, mikä mahdollistaa komponenttien vaihtamisen kokeilu- ja tutkimustarkoituksiin.
* **Rinnakkainen renderöinti**:
  Renderöinti on erillinen säie, joka on irrotettu layoutista responsiivisuuden ylläpitämiseksi.
  Renderer-säie hallitsee muistiaan manuaalisesti välttääkseen roskienkeruun (garbage collection) tauot.
* **Selektorien täsmäytys**:
  Tämä on helposti rinnakkaistettava ongelma.
  Kuten Gecko, Servo tekee selektorien täsmäytyksen erillisessä läpäisyssä flow tree -rakentamisen sijaan, jotta se on helpompi rinnakkaistaa.
* **Rinnakkainen layout**:
  Rakennamme flow tree:n rinnakkaisella DOM-käynnillä, joka kunnioittaa elementtien, kuten floatien, luomia peräkkäisiä riippuvuuksia.
* **Jäsennys**:
  Olemme kirjoittaneet uuden HTML-parserin Rustilla, keskittyen sekä turvallisuuteen että spesifikaation noudattamiseen.
  Emme ole vielä lisänneet spekulatiivista jäsennystä tai rinnakkaisuutta parseriin.
* **Kuvien dekoodaus**:
  Useiden kuvien dekoodaus rinnakkain on suoraviivaista.
* **Muiden resurssien dekoodaus**:
  Tämä on todennäköisesti vähemmän tärkeää kuin kuvien dekoodaus, mutta kaikki, mitä sivun täytyy ladata, voidaan tehdä rinnakkain, esim. kokonaisten tyylitiedostojen jäsentäminen tai videoiden dekoodaus.
  Tyylitiedostot jäsennetään rinnakkain aina kun mahdollista.

## Haasteet

* **Rinnakkaisuutta vastustavat kirjastot**:
  Jotkin tarvitsemamme kolmannen osapuolen kirjastot eivät toimi hyvin monisäieisissä ympäristöissä.
  Fontit ovat olleet erityisen hankalia.
  Vaikka kirjastot olisivat teknisesti säie-turvallisia, säie-turvallisuus saavutetaan usein kirjaston laajuisella mutex-lukolla, mikä heikentää rinnakkaisuusmahdollisuuksiamme.
* **Liian monta säiettä**:
  Jos heitämme maksimaalisen rinnakkaisuuden ja samanaikaisuuden kaikkeen, ylikuormitamme järjestelmän liian monella säieellä.
* **Liian monta avointa tiedostokahvaa**:
  IPC-viestintä vaatii yleensä tiedostokahvan avaamisen järjestelmässä.
  Olemme törmänneet ongelmiin ([#23910](https://github.com/servo/servo/issues/23910), [#33672](https://github.com/servo/servo/issues/33672), [#23905](https://github.com/servo/servo/issues/23906) tiedostokahvojen loppumiseen IPC-mekanismien liiallisen käytön vuoksi.

## JavaScript ja DOM-bindings

Käytämme tällä hetkellä SpiderMonkeyta, vaikka vaihdettavat moottorit ovat pitkän aikavälin, matalan prioriteetin tavoite.
Jokainen web-sisällön prosessi saa oman JavaScript-runtimeensa.
DOM-bindings käyttävät natiivia JavaScript-moottorin API:a XPCOM:n sijaan, ja ne generoidaan automaattisesti WebIDL:n kautta.

## Moniprosessiarkkitehtuuri

Kuten Chromiumissa ja WebKit2:ssa, tarkoituksemme on luoda luotettu upotusprosessi ja useita vähemmän luotettavia web-sisällön prosesseja.
Korkean tason API on IPC-pohjainen, ei-IPC-toteutuksilla testausta ja yksiprosessikäyttötapauksia varten, vaikka odotetaankin, että useimmat vakavat käyttötapaukset käyttävät useita prosesseja.
Moottoriprosessit käyttävät käyttöjärjestelmän sandboxointimahdollisuuksia rajoittaakseen pääsyä järjestelmäresursseihin.

Rustin tyyppijärjestelmä lisää myös merkittävän puolustuskerroksen muistiturvallisuusaukkoja vastaan.
Tämä yksinään ei tee sandboxista vähemmän tärkeää turvallisen koodin, tyyppijärjestelmän virheiden ja kolmannen osapuolen/isäntäkirjastojen puolustamiseksi, mutta se pienentää Servon hyökkäyspintaa merkittävästi verrattuna muihin selainmoottoreihin.
Lisäksi meillä on suorituskykyyn liittyviä huolia joistakin sandboxointitekniikoista (esimerkiksi kaikkien OpenGL-kutsujen välittäminen erilliseen prosessiin).

## I/O ja resurssienhallinta

Verkkosivut riippuvat laajasta valikoimasta ulkoisia resursseja, joilla on monia hakemiseen ja dekoodaukseen liittyviä mekanismeja.
Nämä resursseja välimuistitetaan useilla tasoilla — levylle, muistiin ja/tai dekoodatussa muodossa.
Rinnakkaisessa selainympäristössä nämä resurssit on jaettava samanaikaisille worker-säieille.

Perinteisesti selaimet ovat olleet yksisäieisiä, suorittaen I/O:n "pääsäieellä", jossa suurin osa laskennasta tapahtuu.
Tämä johtaa viiveongelmiin.
Servossa ei ole "pääsäiettä", ja kaikkien ulkoisten resurssien latauksen hoitaa yksi [resource manager] -tehtävä.

[resource manager]: https://github.com/servo/servo/blob/main/components/net/resource_thread.rs

Selaimilla on monia välimuisteja, ja Servon tehtäväpohjainen arkkitehtuuri tarkoittaa, että niitä on todennäköisesti enemmän kuin olemassa olevissa selainmoottoreissa (esim. meillä voi olla sekä globaali tehtäväpohjainen välimuisti että tehtäväkohtainen välimuisti, joka tallentaa tuloksia globaalista välimuistista säästääkseen schedulerin kierroksen).
Servolla tulisi olla yhtenäinen välimuistitarina, säädettävillä välimuisteilla, jotka toimivat hyvin vähämuistisissa ympäristöissä.

## Viitteet

Tärkeää tutkimusta ja kertynyttä tietoa selaimen toteutuksesta, rinnakkaisesta layoutista jne.:

* [How Browsers Work](http://ehsan.github.io/how-browsers-work/#1) - perustason selitys nykyaikaisten verkkoselaimien yleisestä suunnittelusta Gecko-insinööri Ehsan Akhgari
* [More how browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm) - artikkeli, joka on vanhentunut mutta sisältää paljon enemmän yksityiskohtia
* [Webkit overview](https://web.archive.org/web/20150804185551/https://www.webkit.org/coding/technical-articles.html)
* [Fast and parallel web page layout (2010)](https://lmeyerov.github.io/projects/pbrowser/pubfiles/paper.pdf) - Leo Meyerovichin vaikutusvaltainen rinnakkaiset selektorit, layout ja fontit.
  Se suosittelee rinnakkaisten selektorien erottamista rinnakkaisesta cascade:sta muistinkäytön parantamiseksi.
  Katso myös [2013 artikkeli layoutin automatisoinnista](https://lmeyerov.github.io/projects/pbrowser/pubfiles/synthesizer2012.pdf) ja [2009 artikkeli, joka käsittelee spekulatiivista lexing/jäsentämistä](http://lmeyerov.github.io/projects/pbrowser/hotpar09/paper.pdf).
* [Servo layout on mozilla wiki](https://wiki.mozilla.org/Servo/StyleUpdateOnDOMChange)
* [Robert O'Callahan's mega-presentation](http://robert.ocallahan.org/2012/04/korea.html) - Paljon tietoa selaimista
* [ZOOMM paper](https://www.researchgate.net/publication/277679324_ZOOMM) - Qualcommin verkon esihaku ja yhdistetyt selektorit/cascade
* [Strings in Blink](https://chromium.googlesource.com/chromium/src/+/HEAD/third_party/blink/renderer/platform/wtf/text/README.md)
* [Incoherencies in Web Access Control Policies](http://research.microsoft.com/en-us/um/people/helenw/papers/incoherencyAndWebAnalyzer.pdf) - Analyysi document.domainin, cross-origin iframejen ja muun outouden yleisyydestä
* [A Case for Parallelizing Web Pages](https://www.usenix.org/system/files/conference/hotpar12/hotpar12-final58.pdf) -- Sam Kingin palvelinproxy verkkosivujen partitiointiin.
  Katso myös hänen [prosessieristystyönsä, joka raportoi rinnakkaisuuden hyödyistä](https://cseweb.ucsd.edu/~dstefan/cse291-spring21/papers/grier:op.pdf).
* [High-Performance and Energy-Efficient Mobile Web Browsing on Big/Little Systems](https://edge.seas.harvard.edu/sites/g/files/omnuum6351/files/zhu10hpca_0.pdf) Säästä virtaa vaihtamalla dynaamisesti käytettävää ydintä automaattisen työkuorman heuristiikan perusteella
* [C3: An Experimental, Extensible, Reconfigurable Platform for HTML-based Applications](https://web.archive.org/web/20140718031023/http://research.microsoft.com/apps/pubs/default.aspx?id=150010) Microsoft Researchin C#-selainprototyyppi, joka tarjosi samanaikaisen (vaikkakaan ei onnistuneesti rinnakkaistetun) arkkitehtuurin
* [CSS Inline vertical alignment and line wrapping around floats](https://github.com/dbaron/inlines-and-floats) - dbaron jakaa viisautta floateista
* [Quark](http://goto.ucsd.edu/quark/) - Muodollisesti verifioitu selainkernel
* [HPar: A Practical Parallel Parser for HTML](https://web.archive.org/web/20150823220338/https://www.cs.ucr.edu/~zhijia/papers/taco13.pdf)
* [Gecko HTML parser threading](https://web.archive.org/web/20171209054744/https://developer.mozilla.org/en-US/docs/Mozilla/Gecko/HTML_parser_threading)
