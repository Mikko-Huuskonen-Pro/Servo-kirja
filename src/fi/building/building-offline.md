# Offline-käännös

Servon julkaisut tarjoavat `servo-<release-tag>-src-vendored.tar.gz` -artifaktin, joka sisältää Servon lähdekoodin ja kaikki Rust-riippuvuudet vendoroituina.
Tiedoston `.cargo/config.toml` sisältää tarvittavan konfiguraation Servon offline-kääntämiseen vendoroituja riippuvuuksia käyttäen.

## Linux

Katso [Linux-kääntämisohjeet](./linux.md) lisätietoja käännösriippuvuuksien asentamisesta.
Suosittelemme yleensä `./mach build` -komentoa Servon kääntämiseen, mutta se vaatii `uv`:n asennuksen ja Python-riippuvuuksien synkronoinnin `uv sync` -komennolla etukäteen (muuten `./mach` yrittää käyttää verkkoa Python-ympäristön asentamiseen). 
```shell
# online pre-build environment (e.g. building a docker container)
uv sync
# offline build environment
./mach build --profile=production --frozen
```

Jos tämä on vaikeaa, voit kääntää Servon myös `cargo`:lla, jolloin mikä tahansa tuore Python-versio (>= 3.11) riittää (tätä ei testata CI:ssä).
Huomaa, että `./mach build` ottaa `media-gstreamer` -ominaisuuden oletuksena käyttöön — `cargo`:lla sinun täytyy ottaa tämä ominaisuus käyttöön manuaalisesti ja asettaa gstreamer-liittyvät ympäristömuuttujat.
Gstreamer-ominaisuuden vaatimia ympäristömuuttujia ei ole dokumentoitu tässä, mutta dokumentaatioparannukset `./mach build` -koodin pohjalta ovat tervetulleita.

```shell
# Build an offline release build with the production profile.
cargo build --profile=production --frozen
```




## Windows ja macOS

Windowsilla ja macOS:lla `./mach bootstrap` lataa lisäriippuvuuksia Servon kääntämiseen.
Näitä ei tällä hetkellä toimiteta tarballissa. 
Jos offline-käännökset näille alustoille kiinnostavat, osallistuminen on tervetullutta (mutta ota ensin yhteyttä Zulipissa tai GitHub-issueissa).


## Valmiiksi käännetyt SpiderMonkey-artifaktit

Online-käännökset käyttävät oletuksena valmiiksi käännetyitä SpiderMonkey-artifakteja, jotka on hostattu [servo/mozjs](https://github.com/servo/mozjs) -repon GitHub-julkaisuissa.
Jos haluat yksinkertaistaa tai nopeuttaa käännösympäristöäsi, voit ladata nämä artifaktit itse ja käyttää `MOZJS_ARCHIVE=path/to/libmozjs.tar.gz` offline-käännöksissä.
Voit varmistaa ladattujen artifaktien eheyden [GitHub-attestointien] avulla `gh`-työkalulla:

```shell
gh attestation verify path/to/libmozjs.tar.gz -R servo/mozjs
```

[GitHub-attestointien]: https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations
