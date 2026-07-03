# Tehtävien löytäminen

Servon [issue tracker](https://github.com/servo/servo/issues/) sisältää paljon issueita, ja sopivan tehtävän etsiminen voi tuntua ylivoimaiselta.
Tässä on hyödyllisiä vinkkejä issue-tunnisteista, joita kannattaa etsiä:

## Vähemmän monimutkaiset issuet

[E-less-complex](https://github.com/servo/servo/issues/?q=is%3Aissue%20state%3Aopen%20label%3AE-less-complex) -tunniste tarkoittaa, että joku katsoo issuen sopivan uudelle avustajalle.
Näiden tunnisteiden issuen pitäisi sisältää selkeä kuvaus ongelmasta, selkeät vaiheet sen korjaamiseksi ja odotetut varmistusvaiheet.

## Monimutkaisemmat issuet

[E-more-complex](https://github.com/servo/servo/issues/?q=is%3Aissue%20state%3Aopen%20label%3AE-more-complex) -tunniste tarkoittaa, että joku katsoo issuen sopivan hieman kokemusta omaavalle.
Kuten `E-less-complex`-issuet, myös näiden tunnisteiden issuen pitäisi sisältää selkeä kuvaus ongelmasta, selkeät vaiheet sen korjaamiseksi ja odotetut varmistusvaiheet.
Ero kahden tunnisteen välillä on ratkaisuun odotettu vaadittu työmäärä, ja ne voivat vaatia lisätutkimusta.

## Panicien korjaaminen minimoiduilla testitapauksilla

Servo voidaan saada panikoimaan monin tavoin, ja näitä löydetään usein [fuzzaamalla](https://en.wikipedia.org/wiki/Fuzzing) Servoa.
On paljon esimerkkejä issueista, joissa on minimoidut testitapaukset (eli pienin HTML/JS/CSS, jolla ongelma toistuu) ja jotka on merkitty tunnisteilla [I-panic ja C-has-manual-testcase](https://github.com/servo/servo/issues/?q=is%3Aissue%20state%3Aopen%20label%3AI-panic%20label%3AC-has-manual-testcase).

Nämä voivat olla hyviä issueita, koska panic-backtrace voi antaa vihjeitä relevantista koodista, ja voit keskittyä ymmärtämään panicin laukaisevat olosuhteet.

## Testitapausten minimointi

On helppo avata issue siitä, että Servo renderöi verkkosivun väärin, mutta näihin issueihin on usein vaikea reagoida.
On hyvin hyödyllistä etsiä issueita [`C-needs minimized testcase` -tunnisteella](https://github.com/servo/servo/issues/?q=is%3Aissue%20state%3Aopen%20label%3A%22C-needs%20minimized%20testcase%22) ja työskennellä ongelman toistamiseen tarvittavan HTML/CSS/JS:n eristämiseksi.
Katso [opas](guides/minimal-reproducible-test-cases.md) vinkkejä ei-triviaalisten sivujen minimointiin.

## Epäonnistuvien Web Platform Tests -testien diagnosointi

[Web Platform Tests](https://web-platform-tests.org/) (WPT) ovat Servon pääasiallinen automatisoitujen yhteensopivuustestien lähde.
On paljon testejä, joita Servo ei vielä läpäise, ja ne löytyvät [testien metatietohakemistosta](https://github.com/servo/servo/tree/main/tests/wpt/meta).
Jokainen `.ini`-tiedosto kyseisessä hakemistossa vastaa [testin lähdetiedostoa](https://github.com/servo/servo/tree/main/tests/wpt/tests), ja voi olla hyödyllistä valita sinua kiinnostavasta hakemistosta tiedosto ja diagnosoida, mikä estää testin läpäisyn.

Voit myös tutkia kaikkia testejä, joita Servo epäonnistuu, verkkopohjaisessa [wpt.fyi-käyttöliittymässä](https://wpt.fyi/results/?label=master&label=experimental&product=servo&aligned).
Katso [opas](guides/diagnosing-errors/stable-wpt-errors.md) vinkkejä WPT-testien epäonnistumisten syiden tunnistamiseen.
