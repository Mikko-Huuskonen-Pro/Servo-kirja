# Aloitus

Servo toivottaa kaikki avustajat tervetulleiksi.
Verkkomoottorin parissa työskentely voi olla haastavaa ja joskus turhauttavaa, mutta se voi olla myös hyvin palkitsevaa ja hauskaa.
Jos käytät aikaa ja vaivaa osallistumiseen, opit jatkuvasti ja kehityt kehittäjänä ja avoimen lähdekoodin avustajana.
Aloita seuraavasti:

1. Lue loput tästä sivusta saadaksesi perustiedot Servoon avustamisesta.
2. [Hae Servon repositorio](../building/getting-the-code.md) onnistuneesti ja käännä Servo.
2. Luo Servon fork ja opi Gitin perusteet seuraamalla [Git-asetukset](git-setup.md) -luvun ohjeita.
3. Opettele hieman Rustia.
   Verkossa on paljon resursseja, mutta yksi parhaista on virallinen [Learning Rust -dokumentaatio](https://doc.rust-lang.org/stable/#learning-rust).
   Jos tunnet muut imperatiiviset ohjelmointikielet, voit oppia Rustia työskennellessäsi Servon parissa, mutta perustuntemus kielestä on hyödyllinen alussa.
4. [Aseta editorisi](editor-setup.md) niin, että se integroituu `rust-analyzer`-työkalun kanssa.
5. Lue [tyyliohjeemme](style-guide.md) ja tutustu odotuksiin tekemäsi koodin suhteen.
6. Lue ja seuraa [pull requestin tekemisen vaiheet](making-a-pull-request.md).

## Työskentely issuen parissa

Jos haluat työskennellä issuen parissa, jätä siihen kommentti ilmoittaaksesi, että otat sen työn alle.
Näin vältetään päällekkäinen työ samassa issuessa.

Viesti "@servo-highfive assign me" osoittaa issuen sinulle.

Siirry [Servo Starters](https://starters.servo.org/) -sivustolle löytääksesi hyviä tehtäviä aloittelijalle.
Jos kohtaat outoja sanoja tai ammattislangia, tarkista ensin [sanasto](../old/glossary.md).
Jos sopivaa merkintää ei ole, tee pull request ja lisää uusi merkintä sisällöllä `TODO`, jotta voimme korjata sen!

## Tekoälyavustukset

Avustukset eivät saa sisältää suurten kielimallien tai muiden probabilististen työkalujen tuottamaa sisältöä, mukaan lukien mutta ei rajoittuen Copilotiin tai ChatGPT:hen. Tämä käytäntö kattaa koodin, dokumentaation, pull requestit, issuet, kommentit ja kaikki muut Servo-projektin avustukset.

Toistaiseksi suhtaudumme näihin työkaluihin varovaisesti niiden vaikutusten vuoksi — sekä tuntemattomien että havaittujen — projektin terveyteen ja ylläpitotaakkaan. Alue kehittyy nopeasti, joten olemme avoimia käytännön uudelleen arvioinnille myöhemmin, jos esitetään ehdotuksia työkaluista, jotka lieventävät näitä vaikutuksia. Perustelumme ovat seuraavat:

**Ylläpitäjien taakka:** Arvioijat luottavat siihen, että avustajat kirjoittavat ja testaavat koodinsa ennen lähettämistä. Olemme havainneet, että nämä työkalut helpottavat suurten määrien uskottavalta näyttävän koodin tuottamista, jota avustaja ei ymmärrä, jota usein ei ole testattu ja joka ei toimi oikein. Tämä kuormittaa arvioijiemme (jo valmiiksi rajallista) aikaa ja energiaa.

**Oikeellisuus ja turvallisuus:** Vaikka tekoälytyökalujen tuottama koodi näyttäisi toimivan, oikeellisuudesta ei ole takeita eikä merkkejä mahdollisista turvallisuusvaikutuksista. Selainmoottori on rakennettu toimimaan vihamielisissä suoritusympäristöissä, joten kaiken koodin on otettava huomioon mahdolliset turvallisuusongelmat. Avustajilla on suuri rooli näiden asioiden pohtimisessa avustuksia luodessaan — emme voi luottaa tekoälytyökalun tekevän tätä.

**Tekijänoikeuskysymykset:** Julkisesti saatavilla olevat mallit on koulutettu tekijänoikeudella suojatulla sisällöllä sekä vahingossa että tarkoituksella, ja niiden tuotos sisältää usein kyseistä sisältöä sanatarkasti. Koska tämän laillisuus on epävarmaa, avustukset voivat rikkoa tekijänoikeudella suojattujen teosten lisenssejä.

**Eettiset kysymykset:** Tekoälytyökalujen rakentaminen ja käyttö vaatii kohtuuttoman paljon energiaa ja vettä, niiden mallit rakennetaan raskaasti hyväksikäytetyillä työntekijöillä hyväksymättömissä työoloissa, ja niitä käytetään työvoiman heikentämiseen ja irtisanomisten oikeuttamiseen. Emme halua jatkaa näitä haittoja, edes epäsuorasti.

<div class="warning">

Huomaa, että koodin tai muiden avustusten tuottamisen lisäksi tekoälytyökalut voivat joskus vastata Servoa koskeviin kysymyksiisi, mutta olemme havainneet näiden vastausten olevan usein virheellisiä tai hyvin harhaanjohtavia.

Yleisesti ottaen älä oleta tekoälytyökalujen olevan totuuden lähde Servon toiminnasta. Harkitse kysymysten esittämistä [Zulipissa](https://servo.zulipchat.com) sen sijaan.
</div>

### Tekoälykäytännön FAQ

#### Voinko käyttää tekoälytyökaluja kääntäessäni äidinkielestäni englanniksi?

Kyllä.

#### Voinko käyttää tekoälytyökalua auttamaan bugien tai turvallisuusongelmien löytämisessä Servossa?

Kyllä, mutta sinun on varmennettava tekoälytyökalun tuotos ennen issuen avaamista Servo-projektia vastaan.
Projektia vastaan avattujen issuen odotetaan noudattavan projektin tekoälykäytäntöä — sinun on pystyttävä toistamaan ja validoimaan havainnot, et pelkästään luottamaan työkalun tuotokseen.

#### Voinko käyttää tekoälytyökalua ymmärtääkseni Servon koodikantaa paremmin?

Kyllä, mutta katso aiempi varoitus näiden työkalujen luotettavuudesta.

#### Voinko lähettää pull requestin, joka sisältää tekoälytyökalujen tuottamaa koodia?

Et.
Tämä kattaa (mutta ei rajoitu) kaikki generatiiviset tekoälytyökalut, kuten Clauden, Codexin, ChatGPT:n, Cursorin, Geminin ja Windsurfin.

#### Voinko käyttää tekoälytyökalua pull requestin tiivistämiseen?

Et.
Tämä kattaa (mutta ei rajoitu) kaikki generatiiviset tekoälytyökalut, kuten Clauden, Codexin, ChatGPT:n, Cursorin, Geminin ja Windsurfin.
Kuvaile pull requestisi noudattamalla [projektin parhaita käytäntöjä](making-a-pull-request.md#title-and-description).

#### Voinko käyttää tekoälyarvioijaa pull requestilleni?

Kyllä, mutta älä tee sitä.
Tulokset ovat epäluotettavia ja meluisia.

#### Voinko käyttää tekoälytyökalua uuden issuen kuvauksen tai kommentin kirjoittamiseen?

Et.
Tämä kattaa (mutta ei rajoitu) kaikki generatiiviset tekoälytyökalut, kuten Clauden, Codexin, ChatGPT:n, Cursorin, Geminin ja Windsurfin.

#### Mitä tapahtuu, jos rikon tätä tekoälykäytäntöä?

Servon ylläpitäjät pidättävät oikeuden sulkea minkä tahansa pull requestin, joka ei täytä tämän käytännön vaatimuksia.
Kaikki kommentit, kuvaukset ja muut ei-koodiset artefaktit, joita aiemmat FAQ-kohdat eivät salli, on kirjoitettava uudelleen ilman tekoälytyökalujen käyttöä.

## Käyttäytyminen

Servon käyttäytymissäännöt on julkaistu osoitteessa <https://servo.org/coc/>.

## Hallinto

Servon hallinto määritellään **Technical Steering Committee (TSC)** -elimellä, ja se on dokumentoitu [`servo/project`-repositoriossa](https://github.com/servo/project/blob/main/governance/README.md).
