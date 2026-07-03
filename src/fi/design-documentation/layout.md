<!-- TODO: needs copyediting -->

# Layout

Servon nykyinen layout-järjestelmä tunnetaan myös nimellä Layout 2020. Se korvasi [alkuperäisen layout-järjestelmän](https://github.com/servo/servo/wiki/Servo-Layout-Engines-Report), joka tunnettiin myös nimellä Layout 2013.

Layout tapahtuu kolmessa vaiheessa: box tree -rakentaminen, fragment tree -rakentaminen ja display list -rakentaminen.
Kun display list on generoitu, se lähetetään [WebRenderille](https://github.com/servo/webrender) renderöintiä varten.
Aina kun mahdollista puurakentamisen aikana, layout yrittää käyttää rinnakkaisuutta Rayonin avulla.
Tietyt CSS-ominaisuudet estävät rinnakkaisuuden, kuten floatit tai laskurit.
Sama koodi käytetään sekä rinnakkaisessa että peräkkäisessä layoutissa.

## Box Tree

*Box tree* on puu, joka edustaa sisäkkäisiä formatting contexteja [CSS-spesifikaatiossa][formatting-context] kuvatulla tavalla.
Formatting contexteja on erilaisia, kuten block formatting context (block flow:lle), inline formatting context (inline flow:lle), table formatting context ja flex formatting context.
Jokaisella formatting contextilla on eri säännöt laatikoiden sijoittelulle kontekstin sisällä.
Servo edustaa tätä kontekstipuuta sisäkkäisillä enum:eilla, jotka varmistavat, että kontekstin sisältö voi olla vain sellaista sisältöä, jota spesifikaatio kuvaa.

Box tree on vain layout-tilan alkuperäinen esitys, ja yleisesti ottaen seuraava vaihe on ajaa layout-algoritmi box tree:llä ja tuottaa fragment tree.
CSS:n fragmentit ovat tuloksia elementtien jakamisesta box tree:ssä useisiin fragmentteihin rivinvaihtojen, sarakkeiden ja sivutuksen vuoksi.
Lisäksi layout-vaiheessa Servo sijoittaa ja mitoittaa tuloksena olevat fragmentit suhteessa niiden containing blockeihin.
Muunnos tapahtuu yleensä funktiossa `layout(...)` eri box tree -tietorakenteilla.

*Box tree*:n layout *fragment tree*:ksi tehdään rinnakkain, kunnes tullaan puun osaan, jossa on floatit.
Näissä osissa tehdään peräkkäinen läpäisy, ja rinnakkainen layout voi jatkua uudelleen, kun layout-algoritmi siirtyy floatit sisältävän block formatting contextin rajojen yli, joko laskeutumalla riippumattomaan formatting contextiin tai viimeistelemällä float-kontin layoutin.

[formatting-context]: https://drafts.csswg.org/css-display/#formatting-context

## Fragment Tree

Layout-vaiheen tuote on *fragment tree*.
Tässä puussa elementeillä, jotka on jaettu eri paloihin rivinvaihtojen, sarakkeiden tai sivutuksen vuoksi, on fragmentti jokaiselle palalle.
Lisäksi jokainen fragmentti on sijoitettu suhteessa fragmenttiin, joka vastaa sen containing blockia.
Positionoituja fragmentteja varten alkuperäiseen puun sijaintiin jätetään ylimääräinen placeholder-fragmentti `AbsoluteOrFixedPositioned`.
Tätä placeholderia käytetään display listin rakentamiseen oikeassa järjestyksessä CSS:n maalausjärjestyksen mukaan.

## Display List -rakentaminen

Kun layout on luonut *fragment tree*:n, se voi siirtyä renderöinnin seuraavaan vaiheeseen, joka on display listin tuottaminen puulle.
Tässä vaiheessa *fragment tree* muunnetaan [WebRender](https://github.com/servo/webrender) -display listiksi, joka koostuu display list -kohteista (suorakulmiot, viivat, kuvat, tekstijaksot, varjot jne.).
WebRender ei tarvitse suurta valikoimaa display list -kohteita web-sisällön edustamiseen.

Normaalien display list -kohteiden lisäksi WebRender käyttää *spatial node* -puuta muunnosten, scrollattavien alueiden ja sticky-sisällön edustamiseen.
Tämä puu on käytännössä kuvaus siitä, miten post-layout-muunnoksia sovelletaan display list -kohteisiin.
Kun sivua scrollataan, juuren scrollaus-solmun offsetia voidaan säätää ilman välitöntä layoutia.
Samoin WebRender voi soveltaa muunnoksia, mukaan lukien 3D-muunnoksia, web-sisältöön spatial node -tyypillä, jota kutsutaan *reference frame*:ksi.

Leikkaus, olipa se CSS-leikkauksesta tai CSS:n `overflow`-ominaisuuden tuomasta leikkauksesta, käsitellään toisella *clip node* -puulla.
Näille solmuille on myös määritetty *spatial node*:t, jotta leikkaukset pysyvät synkassa muun web-sisällön kanssa.
WebRender päättää, miten parhaiten soveltaa sarjan leikkauksia jokaiseen kohteeseen.

Kun display list on rakennettu, se lähetetään compositorille, joka välittää sen WebRenderille.
