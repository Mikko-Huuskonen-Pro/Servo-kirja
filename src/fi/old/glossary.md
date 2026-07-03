# TODO: docs/glossary.md

<!-- https://github.com/servo/servo/blob/b79e2a0b6575364de01b1f89021aba0ec3fcf399/docs/glossary.md -->

# Sanaston käyttö

Tämä on kokoelma yleisiä termejä, joilla on erityinen merkitys Servo-projektin kontekstissa.
Tavoitteena on tarjota korkean tason määritelmiä ja hyödyllisiä linkkejä lisälukemiseen, eikä täydellistä dokumentaatiota koodin yksittäisistä osista.

Jos Servon koodissa, issue trackerissa, mailing listillä jne. on sana tai lause, joka on hämmentävä, tee pull request, joka lisää sen tähän tiedostoon rungolla `TODO`.
Tämä signaaloi asiantuntijoille lisäämään merkityksellisemmän määritelmän.

# Sanasto

### Compositor ###

Säie, joka vastaanottaa syötetapahtumat käyttöjärjestelmältä ja välittää ne constellationille.
Se vastaa myös web-sisällön valmiiden renderöintien compositoinnista ja niiden näyttämisestä ruudulla mahdollisimman nopeasti.

### Constellation ###

Säie, joka hallitsee joukkoa toisiinsa liittyvää web-sisältöä.
Tätä voi ajatella yksittäisen välilehden omistajana välilehdellisessä web-selaimessa; se kapseloi istuntohistorian, tietää kaikista kehysten puussa olevista kehyksistä ja omistaa pipeline:n jokaiselle sisältyvälle kehykselle.

### Display list ###

Lista konkreettisia renderöintiohjeita.
Display list on layoutin jälkeen, joten kaikilla kohteilla on stacking context -suhteiset pikselisijainnit, ja z-index on jo sovellettu, joten myöhemmät kohteet display listissä ovat aina aiempien päällä.

### Layout thread ###

Säie, joka vastaa DOM-puun layoutoinnista laatikoiden kerroksiin tietylle dokumentille.
Vastaanottaa komentoja script-säikeeltä layoutoida sivu ja joko luo uuden display listin renderer-säikeen käyttöön tai palauttaa sivun layoutin kyselyn tulokset scriptin käyttöön.

### Pipeline ###

Yksikkö, joka kapseloi viestintätavan script-, layout- ja renderer-säikeiden kanssa tietylle dokumentille.
Jokaisella pipelinella on globaalisti yksilöllinen id, jolla siihen pääsee käsiksi constellationista.

### Renderer thread (vaihtoehtoisesti paint thread) ###

Säie, joka kääntää display listin sarjaksi piirtokomentoja, jotka renderöivät liittyvän dokumentin sisällön puskuriin, joka lähetetään sitten compositorille.

### Script thread (vaihtoehtoisesti script task) ###

Säie, joka suorittaa JavaScriptiä ja tallentaa DOM-esityksen kaikista dokumenteista, joilla on yhteinen [origin](https://tools.ietf.org/html/rfc6454).
Tämä säie kääntää constellationilta vastaanotetut syötetapahtumat DOM-tapahtumiksi [spesifikaation mukaisesti](https://w3c.github.io/uievents/), kutsuu HTML-parserin, kun uutta sivun sisältöä vastaanotetaan, ja evaluoi JS:ää tapahtumille kuten timereille ja `<script>`-elementeille.
