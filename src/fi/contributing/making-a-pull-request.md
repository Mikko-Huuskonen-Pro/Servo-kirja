# Pull requestin tekeminen

Avustukset Servoon tai sen riippuvuuksiin tehdään GitHub pull request -muodossa.
Jokainen pull request arvioi ydinavustaja (henkilö, jolla on oikeus landata patcheja), ja se joko landataan pääpuuhun tai siihen annetaan palautetta vaadituista muutoksista.
Kaikkien avustusten pitäisi noudattaa tätä muotoa, myös ydinavustajien.

## Pull request -tarkistuslista

- Haaroita `main`-haarasta ja tarvittaessa rebasaa haarasi `main`-haaraan ennen pull requestin lähettämistä.
  Jos se ei yhdisty siististi `main`-haaraan, sinua voidaan pyytää rebasaamaan muutoksesi.

- Aja `./mach fmt` ja `./mach test-tidy` muutoksellesi.

- Commitit pitäisi olla mahdollisimman pieniä, mutta niin, että jokainen commit on itsenäisesti oikea (eli jokaisen commitin pitäisi kääntyä ja läpäistä testit).

- Commiteihin liittyy [Developer Certificate of Origin](http://developercertificate.org) -allekirjoitus, joka osoittaa, että sinä (ja tarvittaessa työnantajasi) sitoudutte [projektin lisenssin](https://github.com/servo/servo/blob/main/LICENSE) ehtoihin.
  Gitissä tämä on `git commit`-komennon `-s`-valitsin.

- Jos patchiasi ei arvioida, tai tarvitset tietyn henkilön arvioivan sen, voit @-vastata arvioijalle pyytäen arviointia pull requestissa tai kommentissa, tai pyytää arviointia [Servo-chatin](https://servo.zulipchat.com/) kautta.

- Lisää testit, jotka liittyvät korjattuun bugiin tai uuteen ominaisuuteen.
  DOM-muutoksessa tämä on yleensä web platform test; asettelussa reftest.
  Katso [testausopas](https://github.com/servo/servo/wiki/Testing) lisätietoja varten.

## Pull requestin avaaminen

Ensin pushaa haarasi `origin`-remoteen.
Seuraavaksi avaa joko onnistuneen pushin palauttama URL aloittaaksesi pull requestin,
tai vieraile Servo-forkissasi selaimessa ja seuraa käyttöliittymän ohjeita.

```
jdm@pathfinder servo % git push origin issue-12345
Enumerating objects: 43, done.
Counting objects: 100% (43/43), done.
Delta compression using up to 12 threads
Compressing objects: 100% (29/29), done.
Writing objects: 100% (29/29), 5.13 KiB | 5.13 MiB/s, done.
Total 29 (delta 25), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (25/25), completed with 14 local objects.
remote:
remote: Create a pull request for 'issue-12345' on GitHub by visiting:
remote:      https://github.com/jdm/servo/pull/new/issue-12345
remote:
```

![Kuvakaappaus GitHub-käyttöliittymän painikkeesta, jossa lukee "Compare & pull request"](../images/github-open-pr.png)

# Otsikko ja kuvaus

Servossa pull requestien commitit squashataan, ja pull requestin otsikko ja kuvaus käytetään lopullisen commit-viestin luomiseen ([esimerkki](https://github.com/servo/servo/commit/9f4f598f44518d6883257f04f2ed11a8edd732c0)).
On tärkeää, että sekä otsikko että kuvaus kuvaavat muutoksen täysin, koska se tekee commitista hyödyllisen repositorion historian tarkastelijoille ja tekee Servosta terveemmän projektin kokonaisuutena.

Otsikon pitäisi tiiviisti kuvata, mitä muutos tekee. Muutamia vinkkejä:

* Etuliite otsikossa on sen craten nimi pienillä kirjaimilla, jota työstät.
  Esimerkiksi jos muutos koskee "script"-cratea, etuliite on "script: ".
  Jos muutos vaikuttaa useisiin crateihin, tunnista, mikä crate on muutoksen ensisijainen "lähde", tai jätä etuliite pois.
* Otsikot kirjoitetaan imperatiivisena (pyyntö- tai käskylauseena) verbin kanssa.
  Esimerkiksi "layout: Skip box tree construction when possible."
  Vältä geneerisiä verbejä kuten "fix", "correct" tai "improve" ja kuvaile sen sijaan, mitä korjaus tekee.
  Koodia voidaan korjata useita kertoja, mutta viestin pitäisi tunnistaa muutos yksilöllisemmin.
* Etuliitteet kirjoitetaan pienillä kirjaimilla, ja otsikon lopun osan pitäisi capitalisoida vain ensimmäinen sana ja erisnimet (kuten tietorakenteiden nimet tai spesifikaatiokäsitteet kuten WebDriver).
  Epävarmuuden vallitessa seuraa spesifikaatiossa kirjoitettua.
  Älä käytä "Title Casing for Commit Messages" -tyyliä.

Commit-kuvauksen pitäisi:

* Kuvata alkuperäinen ongelma tai tilanne (linkaten tarvittaessa avoimiin bugeihin).
* Kuvata, miten muutos korjaa ongelman, parantaa koodia tai valmistelee seuraavaa muutosta.
* Selittää muutoksen varaukset, kuten uudet epäonnistuvat testit, suorituskyvyn heikkeneminen tai paljastumattomat reunatapaukset.
  Keskustele siitä, miten nämä voidaan käsitellä tulevaisuudessa.
* Olla kirjoitettu usean lauseen kappaleina joko yhtenäisellä rivityksellä (80 merkkiä tai vähemmän) tai ilman rivitystä (GitHub tekee tämän automaattisesti).

Oletuspull request -malline sisältää useita kehotteita; täytä ne korvaamalla alkuperäiset ohjeet.

"Testing"-kehotus on erityisen tärkeä, koska se helpottaa arviointiprosessia.
Se kysyy:
* Onko muuttuvaa koodia jo kattavia automatisoituja testejä?
* Jos ei, sisältääkö pull request uusia automatisoituja testejä?
* Jos ei, mikä estää vähintään yhden uuden testin lisäämisen?

Jos et tiedä vastausta, kirjoita se ylös!
Sen voi aina kirjoittaa uudelleen arvioijan palautteen perusteella.

## Arviointikommenttien käsittely

Lisää uusia commiteja haaraan mieluummin kuin muokkaat olemassa olevia commiteja.
Tämä helpottaa Servon arvioijien tarkastella vain muutoksia, mikä nopeuttaa pull requestien arviointia.

```
git commit -s -m "script: Fix deadlock with cross-origin iframes."
git push origin issue-12345
```

## Yhdistämisristiriitojen käsittely

Kun pull requestissa on yhdistämisristiriitoja, yleisimmät tavat käsitellä ne ovat merge ja rebase.
Älä paina pull requestin "Update branch" -painiketta; se suorittaa mergen ja estää osan Servon CI-toiminnallisuudesta toimimasta oikein.

![Kuvakaappaus GitHub-käyttöliittymän painikkeesta, jossa lukee "Update branch"](../images/github-update-branch.png)

Sen sijaan päivitä ensin paikallinen `main`-haarasi, rebasaa sitten feature-haarasi `main`-haaran päälle ja force pushaa muutokset.

```
git checkout issue-12345
git pull --rebase upstream main
git push -f origin issue-12345
```

Kun rebase kohtaa ristiriitoja, sinun on käsiteltävä ne:

```
jdm@pathfinder servo % git rebase main issue-12345
Auto-merging components/script/dom/bindings/root.rs
CONFLICT (content): Merge conflict in components/script/dom/bindings/root.rs
Auto-merging components/script/dom/node.rs
error: could not apply 932c8d3e97d... script: Remove unused field from nodes.
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```

Seuraa GitHubin dokumentaatiota [ristiriitojen ratkaisemisesta](https://docs.github.com/en/get-started/using-git/resolving-merge-conflicts-after-a-git-rebase).

## Testien ajaminen pull requesteissa

Kun pushaat pull requestiin, GitHub tarkistaa automaattisesti, ettei muutoksissasi ole käännös-, lint- tai tidy-virheitä.

Ajaaksesi yksikkötestejä tai Web Platform Tests -testejä pull requestia vastaan, lisää yksi tai useampi alla olevista tunnisteista pull requestiisi.
Jos sinulla ei ole oikeutta lisätä tunnisteita pull requestiisi, lisää kommentti bugiin pyytäen niiden lisäämistä.

| Tunniste           | Ajaa yksikkötestit | Ajaa web-testit            |
|--------------------|--------------------|----------------------------|
| `T-full`           | Kaikilla alustoilla | Linux                      |
| `T-linux-wpt`      | Linux              | Linux                      |
| `T-macos`          | macOS              | (ei mitään)                |
| `T-windows`        | Windows            | (ei mitään)                |

DevTools-testit ovat kokeellisia eivätkä ole vielä oletuksena käytössä.
On erittäin suositeltavaa, että pull requestit, jotka muokkaavat DevTools-koodia, ajavat ne.
Lisää tätä varten `T-linux-devtools`-tunniste.
Kuten Web Platform Tests -testeissä, jos sinulla ei ole oikeutta lisätä tunnisteita, pyydä kommentilla niiden lisäämistä.
