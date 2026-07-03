# DOM-rajapinnan toteuttaminen

## Osa 1: Web API:n perusasetukset
1. Lue relevantti spesifikaatio.
2. Lisää `.webidl`-tiedosto(t) [tähän hakemistoon](https://github.com/servo/servo/tree/168f7ead152c679ba1e0b8cdddd89e66433b512b/components/script_bindings/webidls) jokaiselle toteutettavalle rajapinnalle. Jos tiedosto on jo olemassa, lisää siihen puuttuvat osat.
3. Jokaiselle rajapinnalle tämä luo traitin nimeltä `{interface_name}Methods`,
   johon pääsee käsiksi komennolla `use crate::dom::bindings::codegen::Bindings::{interface_name}Binding`.
4. Käytä traitia:
   - Lisäämällä vastaavan structin `#[dom_struct]`-attribuutilla
   - Lisäämällä metodeja, joiden runko on `todo!`.
5. Tässä vaiheessa structilla voi olla vain yksi jäsen: `reflector_: Reflector,`
   - Struct pitää dokumentoida linkillä sen rajapintaan spesifikaatiossa,
   - Trait-metodit pitää dokumentoida linkillä niiden määritelmiin rajapinnassa.
   - [Esimerkkitulos](https://github.com/servo/servo/pull/34844/commits/5c842527d89f9c715f27427913a8d5fc6b18d4c7).
5. Palaa spesifikaatioon,
   - etsi structillesi lisäjäseniä.
   - Näitä kutsutaan yleensä „internal sloteiksi” ([esimerkki](https://streams.spec.whatwg.org/#default-reader-internal-slots)),
   tai joksikin, mikä on „associated” rajapinnan kanssa ([esimerkki](https://html.spec.whatwg.org/#message-channels)).
6. Lue [DOM-rajapinnat](https://github.com/servo/servo/blob/4a5ff01e060293721d10289ec56dbd4c58a0969e/components/script/dom/mod.rs).
7. Käyttäen oppimaasi, jokaiselle rajapinnan internal slotille:
   - Lisää sopiva jäsen kohdassa 4 lisättyihin structeihin.
   - Jos tämä vaatii muiden structien tai enumien määrittelyä, niiden pitää derivoida `JSTraceable` ja `MallocSizeOf` ([esimerkki](https://github.com/servo/servo/pull/34844/commits/7d73370b0c41a1b00f4b25b7e1b8bf9b67430708#diff-2e7f6e100fdbd73318de2dda9b3d3883700be9ebfd028d4412a207e93cb02892R53)).
   - Kaikki yllä lisätyt `JSTraceable`-structit, jotka sisältävät jäseniä, jotka täytyy rootata koska ne ovat joko [JS-arvoja](https://github.com/servo/mozjs/blob/87cabf4e9ddf9fafe19713a3d6bc8c5e6105544c/mozjs/src/gc/collections.rs#L94) tai [DOM-objekteja](https://github.com/servo/servo/blob/9887ad369d65eb362db21c778ae5f00aad74db6c/components/script/dom/bindings/root.rs#L5), pitää merkitä `#[cfg_attr(crown, crown::unrooted_must_root_lint::must_root)]`. [Esimerkki](https://github.com/gterzian/servo/blob/b7688fe916d105ae9023cd2429068f16ecba3574/components/script/dom/readablestreamdefaultcontroller.rs#L120), jossa lintti tarvitaan `JSVal`-jäsenen vuoksi.
   - Jos tällainen struct assignataan muuttujaan, tee `impl js::gc::Rootable` structille ja käytä `rooted!` rootataksesi muuttujan ([esimerkki](https://github.com/servo/servo/pull/34844/commits/94867eec21e06d59c5479bdaa92ef422bc7b21f9)).
   - Kaikki tämä voidaan muuttaa myöhemmin, joten käytä parasta arvostelukykyäsi tässä vaiheessa.
   - Lisää metodeja [rakentamiseen](https://github.com/servo/servo/blob/4a5ff01e060293721d10289ec56dbd4c58a0969e/components/script/dom/mod.rs#L91) (älä sekoita Web API:n osana olevaa `Constructor`-metodia).
   - [Esimerkkitulos](https://github.com/servo/servo/pull/34844/commits/7d73370b0c41a1b00f4b25b7e1b8bf9b67430708).

## Osa 2: Ensimmäisen luonnoksen kirjoittaminen
1. Jokaiselle osan 1 kohdassa 3 mainitun bindings-traitin metodille:
   - Yleensä seuraa spesifikaation rakennetta: jos metodi kutsuu toista nimettyä algoritmia, toteuta se erillisenä structin yksityisenä metodina, jota trait-metodi kutsuu. Jos myöhemmin huomaat, että tätä yksityistä metodia voi käyttää muista structeista, tee siitä `pub(crate)`.
   - Jokaiselle spesifikaation algoritmivaiheelle:
       - Kopioi rivi spesifikaatiosta.
       - Toteuta spesifikaatio koodissa (voi vaatia useamman rivin ja lisäkommentteja).
2. Huom: tietyt asiat tarvitaan usein algoritmin osana:
   - `JSContext` tai `CurrentRealm`, jotka ovat toisensa poissulkevia, saadaan generoidun trait-metodin argumenttina [tämän konfiguraatiotiedoston](https://github.com/servo/servo/blob/4a5ff01e060293721d10289ec56dbd4c58a0969e/components/script_bindings/codegen/Bindings.conf) avulla
   - `GlobalScope`: saadaan `self.global()`-kutsulla `dom_struct`-structilla tai `GlobalScope::from_current_realm`-kutsulla
   - On parasta hakea ne mahdollisimman aikaisin, esimerkiksi trait-metodin toteutuksen alussa, ja välittää ne alaspäin ref:inä (useimmissa tapauksissa `JSContext`/`CurrentRealm` täytyy olla `&mut`-viitteen takana)
   - On suositeltavaa, että `JSContext`/`CurrentRealm` on ensimmäinen argumentti, sen jälkeen `&GlobalScope` tarvittaessa, ja muut tarvittavat argumentit perässä
2. Tämän pitäisi antaa täydellinen ensimmäinen luonnos.

## Osa 3: Testien ajaminen ja bugien korjaaminen
1. Nyt on aika tunnistaa, mitä [WPT-testiä](../testing.md) ajetaan ensimmäistä luonnosta vasten. Löydät ne [täältä](https://github.com/servo/servo/tree/168f7ead152c679ba1e0b8cdddd89e66433b512b/tests/wpt).
    - Tämä voi vaatia niiden ottamista käyttöön [tällä konfiguraatiolla](https://github.com/servo/servo/blob/168f7ead152c679ba1e0b8cdddd89e66433b512b/tests/wpt/include.ini).
 2. Testi voi epäonnistua, koska:
     - Koodissa on bugi. Nämä pitää korjata.
     - Testi käyttää muita API:ja, joita ei vielä tueta (yleensä `ERROR`).
3. Bugit pitää korjata. Copilotista on tässä vähän hyötyä.
4. Odotetut epäonnistumiset voidaan merkitä sellaisiksi [tässä](../testing.md#updating-web-platform-test-expectations) kuvatussa prosessissa.
5. Tämä osa on valmis, kun odottamattomia testituloksia ei enää ole.
6. Joskus reviewaajan neuvosta voit tehdä issuen ja kuvata epäonnistumisen, jota et voi korjata, merkitä testin epäonnistumiseksi ja jättää sen seuraavaan työvaiheeseen.

## Osa 4: Refaktorointi ja lopullisen reviewn pyytäminen
1. Voit pyytää reviewa milloin tahansa, jos olet jumissa, mutta nyt on aika katsoa koodia viimeisen kerran ja päättää, haluatko refaktoroida jotain.
2. Jos olet tyytyväinen, nyt on aika pyytää lopullinen review.
3. Onnittelut, toteuttamasi uusi Web API pitäisi pian yhdistää.
