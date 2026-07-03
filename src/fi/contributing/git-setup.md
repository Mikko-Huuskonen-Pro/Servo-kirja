# Git-asetukset

Jos olet uusi Gitin tai hajautetun versionhallinnan parissa, on erittäin suositeltavaa käyttää aikaa perusteiden opetteluun ennen jatkamista.
Verkossa on monia erinomaisia resursseja Gitin oppimiseen.
Esimerkiksi Zulip-projekti ylläpitää [hyvin kattavaa opasta](https://zulip.readthedocs.io/en/latest/git/index.html) Gitin käyttöön avoimen lähdekoodin projektiin avustamisessa.
Tiedot ovat usein hyvin relevantteja myös Servon parissa työskentelyyn.
Gitin hallinta on taito, josta on hyötyä monenlaisissa ohjelmistokehitystehtävissä, ja siihen käytetty aika on sen arvoista.

Kun kloonasit Servon alun perin, upstream-Servo-repositorio osoitteessa `https://github.com/servo/servo` oli upstreamisi.
Voit pitää tuon konfiguraation, mutta suositeltu työnkulku on seuraava:

1. [Forkkaa](https://github.com/servo/servo/fork) upstream-Servo-repositorio.
2. Checkoutaa klooni juuri forkkaamastasi Servo-kopiosta.
   ```sh
   git clone --depth 10 https://github.com/<username>/servo.git
   ```
   Huomaa, että `--depth 10` -argumentit hylkäävät suurimman osan Servon commit-historiasta paremman suorituskyvyn vuoksi.
   Ne voidaan jättää pois.
3. Lisää uusi remote nimeltä `upstream`, joka osoittaa `servo/servo`-repoon.
   ```sh
   git remote add upstream https://github.com/servo/servo.git
   ```

## Uuden muutoksen aloittaminen

Kun haluat työskennellä uuden muutoksen parissa, älä tee sitä `main`-haarassa, koska siellä haluat pitää upstream-repositorion kopion ajan tasalla.
Tee työsi sen sijaan haarassa.

1. Päivitä `main`-haarasi upstreamin uusimpiin muutoksiin.
   ```sh
   git checkout main
   git pull origin main
   ```
2. Luo uusi haara `main`-haaran pohjalta.
    ```sh
    git checkout -b issue-12345
    ```
3. Tee muutoksesi ja commitoi ne kyseiselle haaralle.
   Muista myös [allekirjoittaa jokainen commit](https://developercertificate.org/)!
   ```sh
   git commit --signoff -m "script: Add a stub interface for MessagePort"
   ```

Seuraavaksi haluat todennäköisesti [tehdä pull requestin](making-a-pull-request.md) muutoksillasi.
