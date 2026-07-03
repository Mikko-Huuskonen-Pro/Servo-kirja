<!-- TODO: needs copyediting -->

# Servon tyylisysteemin yleiskatsaus

Tämä dokumentti tarjoaa yleiskatsauksen Servon tyylisysteemistä.
Laajempia yksityiskohtia varten katso [style doc comments][style-doc] tai wikin [Styling Overview][wiki-styling-overview], joka sisältää keskustelun Boris Zbarskyn ja Patrick Waltonin välillä siitä, miten tyylien jakaminen toimii.

<a name="selector-impl"></a>
## Selektoritoteutus

Stylo-yhteensopivuuden varmistamiseksi (projekti, joka integroi Servon tyylisysteemin Geckoon) selektorien täytyy olla yhdenmukaisia.

Yhdenmukaisuus on toteutettu [selectors' SelectorImpl][selector-impl] -kohdassa, joka sisältää logiikan pseudo-elementtien ja muiden pseudo-luokkien jäsentämiseen [puurakenteellisten pseudo-luokkien][tree-structural-pseudo-classes] lisäksi.

Servo laajentaa selektoritoteutuksen trait:ia salliakseen muutamien asioiden jakamisen Stylon ja Servon välillä.

Servon pääasiallinen toteutus (jota käytetään tavallisissa käännöksissä) on [SelectorImpl][servo-selector-impl].

<a name="dom-glue"></a>
## DOM-liima

DOM:n, layoutin ja tyylin pitäminen eri moduuleissa vaatii muutamia trait:eja.

Stylen [`dom`-trait:t][style-dom-traits] (`TDocument`, `TElement`, `TNode`, `TRestyleDamage`) ovat pääasiallinen "muuri" layoutin ja tyylin välillä.

Layoutin [`wrapper`][layout-wrapper]-moduuli varmistaa, että layout-trait:eilla on vaaditut trait:t toteutettuina.

<a name="stylist"></a>
## Stylist

[`stylist`][stylist]-rakenne sisältää kaikki selektorit ja laitteen ominaisuudet tietylle dokumentille.

Tyylitiedostojen CSS-säännöt muunnetaan [`Rule`][selectors-rule]-objekteiksi.
Ne sijoitetaan sitten [`SelectorMap`][selectors-selectormap] -rakenteeseen pseudo-elementin (katso [`PerPseudoElementSelectorMap`][per-pseudo-selectormap]), tyylitiedoston originin (katso [`PerOriginSelectorMap`][per-origin-selectormap]) ja prioriteetin (katso `normal`- ja `important`-kentät [`PerOriginSelectorMap`][per-origin-selectormap] -rakenteessa) mukaan.

Tämä rakenne luodaan käytännössä kerran per [pipeline][docs-pipeline], vastaavassa LayoutThread:ssä.

<a name="properties"></a>
## `properties`-moduuli

[properties module][properties-module] on mako-malli.
Sen monimutkaisuus johtuu koodista, joka tallentaa ominaisuuksia, [`cascade`-funktiosta][properties-cascade-fn] ja palautetun arvon laskentalogiikasta, joka on altistettu pääfunktiossa.

[style-doc]: https://doc.servo.org/style/index.html
[wiki-styling-overview]: https://github.com/servo/servo/wiki/Styling-overview
[selector-impl]: https://doc.servo.org/selectors/parser/trait.SelectorImpl.html
[servo-selector-impl]: https://doc.servo.org/style/servo/selector_parser/struct.SelectorImpl.html
[tree-structural-pseudo-classes]: https://www.w3.org/TR/selectors4/#structural-pseudos
[style-dom-traits]: https://doc.servo.org/style/dom/index.html
[layout-wrapper]: https://doc.servo.org/layout/wrapper/index.html
[stylist]: https://doc.servo.org/style/stylist/struct.Stylist.html
[selectors-selectormap]: https://doc.servo.org/style/selector_map/struct.SelectorMap.html
[selectors-rule]: https://doc.servo.org/style/stylist/struct.Rule.html
[per-pseudo-selectormap]: https://doc.servo.org/style/selector_parser/struct.PerPseudoElementMap.html
[per-origin-selectormap]: https://doc.servo.org/style/stylist/struct.PerOriginSelectorMap.html
[docs-pipeline]: https://book.servo.org/old/glossary.html#pipeline
[properties-module]: https://doc.servo.org/style/properties/index.html
[properties-cascade-fn]: https://doc.servo.org/style/properties/cascade/fn.cascade.html
