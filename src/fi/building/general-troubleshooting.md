# Yleinen vianmääritys

<div hidden>

Katso [tyyliopas](../contributing/style-guide.md#error-messages) ohjeisiin virheilmoitusten muotoilusta.
</div>

Jos Servon kääntämisessä ilmenee ongelmia, joita ei ole listattu muilla sivuilla, kokeile ensin alla olevia vaiheita:

1. **Varmista, että sinulla on kaikki listatut käännösvaatimukset**.
   Erityisesti jos käytät harvinaista Linux-jakelua tai muuta Unix-tyyppistä järjestelmää, sinun täytyy ehkä selvittää tietyn riippuvuuden oikea nimi järjestelmässäsi.
2. **Tarkista uudelleen, että käännösvaatimukset on asennettu** ja katso [riippuvuusversiot](#riippuvuusversiot).
3. **Aja `./mach boostrap` tai `.\mach boostrap` Windowsilla.**
   Joskus Servon kääntämiseen tarvittavat työkalut tai riippuvuudet muuttuvat.
   Komennon voi ajaa useamman kerran turvallisesti.
   Varmista, ettei suorituksen aikana ilmoitettu virheitä.
   Jos olet asentanut muita riippuvuuksia manuaalisesti, sinun täytyy ehkä ajaa `./mach bootstrap --skip-platform`.
4. **Päivitä ympäristösi**. Tämä voi tarkoittaa:
   - Shellin uudelleenkäynnistystä
   - Uloskirjautumista ja takaisin sisäänkirjautumista
   - Tietokoneen uudelleenkäynnistystä
   
## Riippuvuusversiot

- `curl --version` tulostaa version kuten 7.83.1 tai 8.4.0
  - Windowsilla kirjoita `curl.exe --version` PowerShellin `Invoke-WebRequest` -aliasin välttämiseksi
- `uv --version` tulostaa version **0.4.30 tai uudemman**
  - Servon `mach`-käännöstyökalu riippuu `uv`:sta kiinnitetyn Python-version (repossa `.python-version`-tiedostossa) tarjoamiseen ja paikallisen virtuaaliympäristön (`.venv`-hakemisto) luomiseen, johon Python-riippuvuusmoduulit asennetaan.
  - Jos järjestelmässä on jo vaaditun Python-version asennus, `uv` linkittää siihen levytilan säästämiseksi.
  - Jos versiot eivät täsmää tai Pythonia ei ole asennettuna, `uv` lataa tarvittavat binäärit.
  - Ulkoisesti hallitun Python-asennuksen käyttöä `mach`-skriptin ajamiseen ei tällä hetkellä tueta.
- `rustup --version` tulostaa version kuten 1.26.0
- **Windows**: `choco --version` tulostaa version kuten 2.2.2
- **macOS**: `brew --version` tulostaa version kuten 4.2.17

## Et ole yksin!

Jos Servon kääntämisessä on ongelmia, joita et ratkaise, voit aina pyytää apua [build issues](https://servo.zulipchat.com/#narrow/stream/263398-general/topic/Build.20Issues) -keskustelussa Zulipissa.
