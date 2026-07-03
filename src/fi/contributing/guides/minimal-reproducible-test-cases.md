# Minimaalinen toistettava testitapaus

Jotta voit tunnistaa tarkalleen, mikä sivulla menee pieleen, on erittäin tärkeää osata luoda minimaalinen toistettava testitapaus.
Vaikka et aikoisikaan korjata ongelmaa itse, testitapauksen tarjoaminen helpottaa muiden korjaustyötä huomattavasti ja säilyttää epäonnistumisen, vaikka alkuperäinen sivu muuttuisi.
Testitapauksen luominen on melko systemaattista prosessia, jonka voi tehdä jopa hyvin uusi web-alustan kehittäjä.
Minimaalisesti toistettava testitapaus on lähes aina ensimmäinen askel ongelman korjaamisessa, ja ne voidaan usein helposti muuntaa Web Platform Testeiksi.

## Peruslähestymistapa

Peruslähestymistapa minimaalisen toistettavan testitapauksen luomiseen on poistaa vähitellen tarpeetonta sisältöä sivulta, kunnes jäljellä on vain ongelmallinen osa.
Tämä tarkoittaa, että alkuperäinen layout-ongelma, DOM-virhe tai kaatuminen tapahtuu edelleen, vaikka lähdekoodi on paljon pienempi.
On mahdollista, ettei virhe näytä tai toimi täsmälleen samalta testitapauksessa, mutta sen pitäisi silti tuottaa huonoja tuloksia verrattuna muihin selaimiin, kaatua tai tuottaa virheen.
Tärkeä vaihe on vertailla tuloksia useampaan muuhun selainmoottoriin nähdäksesi, onko kyseessä oikeasti spesifikaatio-ongelma.
On suositeltavaa ajaa testitapaus myös Chromessa, Firefoxissa ja WebKit-pohjaisessa selaimessa kuten Safarissa.

## Minimointi

Luodaksesi minimaalisen toistettavan testitapauksen tarvitset ensin kopion sivusta tietokoneellesi.
Chromessa tallenna ongelmallinen sivu valinnalla „Webpage, Complete.”
Näin varmistat, että kaikki kuvat, CSS- ja JavaScript-tiedostot tallentuvat myös tietokoneellesi.
Seuraavaksi lataa tallennettu HTML-tiedosto Chromessa ja Servossa varmistaaksesi, että sivu toimii edelleen ja bugi näkyy Servossa.
Nyt on aika pienentää!
Voit käyttää muutamia tekniikoita:

- Etsi sivun osa, joka ei liity ongelmaan, kuten ylä- tai alatunniste.
  Poista ne ladatusta HTML-tiedostosta Web Inspectorissa, esimerkiksi korostamalla elementit ja painamalla *Delete*-näppäintä.
  Tallenna sivu uudelleen ja varmista, että bugi on edelleen olemassa lataamalla uudelleen tallennettu sivu.
  Jatka!
- Jos ongelma on layout-ongelma, kokeile poistaa kaikki sivulla ladattu JavaScript.
  Jos JavaScriptin poistaminen poistaa ongelman, peru muutos ja kokeile poistaa muuta JavaScriptiä.
- Kokeile poistaa viittaukset ulkoisiin tyylitiedostoihin.
  Jos tyylitiedoston poistaminen poistaa myös bugin, upota tyylitiedosto inline-muotoon ja poista vähitellen asiaankuulumattomia sääntöjä.
  Tarvittaessa voit käyttää tyyppistä binääristä eliminointia tyylisäännöille.
- Korvaa ladatut kuvat yksinkertaisella image-tagilla, jossa on `width` ja `height`.
  Ulkoisen resurssin lataamatta jättäminen helpottaa testitapauksen ymmärtämistä huomattavasti.
- Kun sivulla on vain muutama elementti jäljellä, kokeile poistaa `class`-, `id`- tai muita tarpeettomia attribuutteja.

Jatka pienentämistä, kunnes testitapaus on mahdollisimman pieni.
Usein tämän aikana bugin luonne selviää, ja olet jo puolivälissä korjausta.

## Lithium

[Lithium](https://github.com/MozillaSecurity/lithium) on työkalu yllä kuvatun prosessin automatisointiin.
Se toimii erityisen hyvin kaatumisten kanssa ja kestää ei-deterministiset bugit.
