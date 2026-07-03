# Käännöksen tila

Epävirallinen suomenkielinen käännös [Servo Book](https://book.servo.org) -teoksesta.

## Rakenne

- `src/` — alkuperäiset englanninkieliset tiedostot
- `src/fi/` — suomenkieliset käännökset (sama polku ja tiedostonimi)
- `book-fi.toml` — suomenkielisen version mdBook-asetukset

## Kotisatama / oppiminen

[Kotisatama-repon](https://github.com/Mikko-Huuskonen-Pro/Kotisatama) `oppiminen/`-hakemisto sisältää **tiivistelmiä ja oppimispolkuja**, ei suoria kirjaluokkoja. Sitä käytetään terminologian lähteenä (`sanasto.md`), ei kopioitavaksi sisällöksi.

## Rakentaminen

```bash
cargo install mdbook --vers '^0.5' --locked
cargo install mdbook-mermaid --vers '^0.17' --locked
mdbook build -d book-fi.toml
```

## Edistyminen

| Osio | Tila |
|------|------|
| Trying Servo | valmis |
| Building | valmis |
| Embedding Servo | valmis |
| Contributing | valmis |
| Maintainer Guides | valmis |
| Design Documentation | valmis |
| Pre-book docs | valmis |

Kaikki `src/`-markdown-tiedostot on käännetty `src/fi/`-hakemistoon.
