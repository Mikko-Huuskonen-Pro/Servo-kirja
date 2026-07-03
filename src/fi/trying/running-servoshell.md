# Servoshellin käyttö

Oletetaan, että olet hakemistossa, jossa `servo` sijaitsee. Voit ajaa servoshellin komennolla:

```sh
$ ./servo [url] [options]
```

Käytä `--help` nähdäksesi käytettävissä olevat komentorivivalinnat:

```sh
$ ./servo --help
```

### Kokeellisten web-alustan ominaisuuksien käyttöönotto

Servolla on käynnissä tuki monille web-alustan ominaisuuksille.
Jotkut eivät ole vielä valmiita oletuksena käyttöön, mutta voit kokeilla niitä pref-asetuksella.
Luettelo näistä ominaisuuksista ja niitä käyttöönotosta löytyy kohdasta [Kokeelliset ominaisuudet](../design-documentation/experimental-features.md).
Lisäksi voit ottaa käyttöön hyödyllisen osajoukon näistä ominaisuuksista `--enable-experimental-web-platform-features` -komentoriviparametrilla tai servoshellin käyttöliittymän kautta.

## Pikanäppäimet

- **Ctrl**+`Q` (⌘Q macOS:llä) sulkee servoshellin
- **Ctrl**+`L` (⌘L macOS:llä) kohdistaa osoitepalkkiin
- **Ctrl**+`R` (⌘R macOS:llä) lataa sivun uudelleen
- **Alt**+`←` (⌘← macOS:llä) siirtyy historiassa taaksepäin
- **Alt**+`→` (⌘→ macOS:llä) siirtyy historiassa eteenpäin
- **Ctrl**+`=` (⌘= macOS:llä) suurentaa sivun zoomia
- **Ctrl**+`-` (⌘- macOS:llä) pienentää sivun zoomia
- **Ctrl**+`0` (⌘0 macOS:llä) palauttaa sivun zoomin
- **Esc** poistuu koko näytön tilasta

## Vianmääritys

servoshellin pitäisi toimia useimmilla järjestelmillä ilman erillisiä riippuvuuksia.
Jos olet **Linuxilla** ja servoshell ilmoittaa puuttuvasta jaetusta kirjastosta, varmista että seuraavat paketit on asennettu:

* `GStreamer` ≥ 1.18
* `gst-plugins-base` ≥ 1.18
* `gst-plugins-good` ≥ 1.18
* `gst-plugins-bad` ≥ 1.18
* `gst-plugins-ugly` ≥ 1.18
* `libXcursor`
* `libXrandr`
* `libXi`
* `libxkbcommon`
* `vulkan-loader`
