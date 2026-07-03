Servo-kirja
===========

Epävirallinen suomenkielinen käännös [The Servo Book](https://book.servo.org) -teoksesta.

**Julkaistu versio:** <https://mikko-huuskonen-pro.github.io/Servo-kirja/>

Alkuperäinen englanninkielinen kirja: <https://book.servo.org>

**<https://book.servo.org>** (englanninkielinen alkuperä)

## Suomenkielisen version rakentaminen

```sh
$ cargo install mdbook --vers '^0.5' --locked
$ cargo install mdbook-mermaid --vers '^0.17' --locked
$ cp book-fi.toml book.toml && mdbook build && git checkout book.toml
```

Tai Nixillä:

```sh
$ nix-shell --run 'cp book-fi.toml book.toml && mdbook build'
```

## Englanninkielinen versio

```sh
$ mdbook serve --open
```

**Kirja on työn alla!**
Sisällysluettelossa \* merkitsee lukuja, jotka on äskettäin lisätty tai tuotu vanhemmasta dokumentaatiosta ja jotka tarvitsevat vielä oikoluvun tai uudelleentyöstön.

Katso [TRANSLATION_STATUS.md](TRANSLATION_STATUS.md) käännöksen tilasta.

### Uuden sivun lisääminen

1. Luo uusi markdown-sivu haluttuun `src/fi/`-alikansioon
2. Lisää linkki [src/fi/SUMMARY.md](src/fi/SUMMARY.md) -tiedostoon
