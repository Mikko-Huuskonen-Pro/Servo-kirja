# Script
Tämän dokumentin pitäisi tuoda sinut ajan tasalle Script Threadistä, rootingista ja Gc:stä. Se ei millään tavoin ole täydellinen.

Servo on ainutlaatuinen siinä, että se käyttää roskienkeruuta (garbage collection) joihinkin asioihin, jotka eivät ole ilmeisiä.
Esimerkiksi jokainen DOM-objekti (struct, jossa on `#[dom_struct]`) on SpiderMonkeyn roskienkeruun omistama.
Tämä edellyttää, että moottorin API:t, jotka vuorovaikuttavat näiden objektien kanssa, ovat turvallisia, kun roskienkeruu voi tapahtua monissa ohjelman kohdissa.
Vaikka [roskienkeruujärjestelmä](https://firefox-source-docs.mozilla.org/js/gc.html) (GC) on monimutkainen ja sillä on useita tiloja, voimme olettaa, että aina kun GC ajetaan, Rust-ohjelma ei aja.

## Rooting ja rooted-tyypit

Tärkeä osa roskienkeruuta on objektigraafin juurten määrittäminen.
Juuri kertoo roskienkeruulle kaksi asiaa:
1. Älä poista mitään arvoa, johon tästä juuresta on transitiivisesti pääsy.
2. Jos roskienkeruu siirtää tämän juuritetun arvon muistissa, kaikki osoittimet tähän arvoon päivitetään.

Servon koodissa juuret luodaan automaattisesti käyttämällä `DomRoot<T>`, `Root<T>` ja `Rooted<T>` -tyyppejä.
Katso [moduulin dokumentaatio](https://doc.servo.org/script/dom/bindings/root/index.html) lisätietoja eroista. Nämä tyypit toteuttavat `Deref`:in, joten voimme käsitellä niitä
tavallisina viitteinä.

Seuraavat esimerkit sisältävät yksinkertaistuksia oikeista Servo-koodimalleista helpottamaan ymmärtämistä.
Oletetaan seuraava koodi, joka määrittelee `Kittens`-tyypin, joka sisältää osoittimia `Cat`-tyyppeihin, ja kaikki nämä ovat roskienkeruun omistamia:
```rust
#[dom_struct]
struct Kittens {
    children: Vec<Dom<Cat>>
}

fn play_with_kittens(kittens: &Kittens) {
    let children = & cats.children;
    play_with(children)
}
```

Ilman rootingia GC voisi keskeyttää ohjelman sen jälkeen, kun olemme saaneet viitteen children:iin, siirtää `Kittens`-structin ympäriinsä
ja meillä olisi virheellinen pääsy leikkiessämme niiden kanssa!
Tämä pätee, vaikka Kittens olisi juuritettu jollakin muulla mekanismilla aiemmin kutsuketjussa.
Muista, että roskienkeruu ei voi muuttaa paikallisen muuttujamme 'children' osoittimia, jos se ei tiedä niistä.

Siksi meidän täytyy muuttaa koodi muotoon
```rust
#[dom_struct]
struct Kittens {
    children: Vec<Dom<Cat>>
}

fn play_with_kittens(cats: &Kittens) {
    let children: Vec<DomRoot<Cat>> = cats.children.iter().map(|cat| cat.as_rooted()).collect();
    play_with(children)
}
```
Oletamme, että kissaviitteen hankkiminen ja juurittaminen on yksi atomitoiminto.
Juuren poisto (käänteinen operaatio) tapahtuu automaattisesti `DomRoot<T>`:n Drop-toteutuksen kautta.

Varmistaakseen, että web API -toteutukset Servossa ovat turvallisia, moottorikoodi on konservatiivinen ja palauttaa yleensä juuritettyjä tyyppejä (kuten `DomRoot<Node>`).
Tämä on hyväksyttävä kompromissi turvallisuuden ja suorituskyvyn välillä, koska juurittaminen ja juuren poisto ovat tehokkaita; ne käytännössä vain push:avat ja pop:avat vektorin elementtejä.

## Crown ja CanGc
On helppoa unohtaa juurittamiseen liittyvät huolenaiheet toteuttaessa uusia API:ja.
Crown on vastaus (jos työskentelet Script-asioissa, sinun tulisi ajaa `./mach build --use-crown` varmistaaksesi, että se tarkistaa asiat).
Pohjimmiltaan Crown vain tarkistaa, ettei juurittamista ole unohdettu, ja joskus näet koodissa tiettyjä linttejä, jotka viittaavat crown:iin, kuten `#[cfg_attr(crown, allow(crown::unrooted_must_root))]`. Nämä käytännössä poistavat crown-tarkistuksen käytöstä ja niitä tulisi käyttää vain hyvin erityisissä olosuhteissa.

Mutta on toinenkin palanen palapelissä, joka liittyy sivuttain. Vaikka juurittaminen antaa meille tavan pakottaa GC:n käyttäytymään kunnolla Rust-osoittimiemme kanssa, meillä on myös Rust RefCell:it. Vaikka näiden osoittimet käsitellään nyt oikein GC:n toimesta, GC:n täytyy lainata `RefCell`:iä testatakseen saavutettavuutta ja ohjatakseen osoitinta uudelleen.
Mutta voimme lainata `RefCell`:iä vain ei samanaikaisesti. Mitä tapahtuu, jos GC keskeyttää ohjelman, kun meillä on cell-laina aktiivisena?

```rust
struct CatCarrier {
    cat: RefCell<Dom<Cat>>,
}

fn some_function(carrier: &CatCarrier) {
    let mutable_cat = carrier.inner_cat.borrow_mut().as_rooted();
    play_with_cat_mutably(mutable_cat);
    cleanup_playspace();
}
```
Oletetaan nyt, että `cleanup_playspace` voi kutsua GC:tä. Silloin voi tapahtua, että
- Pidämme mutable-lainaa kissalle carrier:in sisällä.
- Siivoamme kissan jälkeen.
- GC keskeyttää ja yrittää jäljittää juuritetun `mutable_cat`:in lainaamalla RefCell:iä.
- Panikoi, koska RefCell:iä voi lainata mutably vain kerran!

Kutsumme tätä borrow hazard -tilanteeksi. Estääksemme tämän, meillä on seuraava ratkaisu.

```rust
fn some_function(carrier: &CatCarrier, can_gc: CanGc) {
    {
        let mutable_cat = carrier.inner_cat.borrow_mut().as_rooted();
        play_with_cat_mutably(mutable_cat);
    }
    cleanup_playspace(can_gc);
}
```

Nyt tämä esimerkki ei ole täydellinen ja on täällä vain seuraavan pointin havainnollistamiseksi.
CanGC (joka on triviaali tyyppi ja helposti kopioitavissa) otettiin käyttöön, ei muodollisena borrow hazard -estona vaan muistutuksena ohjelmoijalle. Se sanoo:
Ole varovainen, jos haluat mutable-lainan, ettei se ylitä funktion rajoja, joka ottaa `CanGc`:n.

Lisätietoja siitä, miten borrow hazard -tilanteet tunnistetaan ja käsitellään, on [täällä](borrow_hazard.md).
Haluamme käyttää tätä vain motivoivana esimerkkinä seuraavalle osiolle `&JSContext`
ja `&mut JSContext`.

Käynnissä oleva esimerkkimme näyttäisi sitten suunnilleen tältä:

```rust
fn some_function(cats: &Kittens, can_gc: CanGc) {
    let children: Vec<DomRoot<Cat>> = cats.children.iter().map(|cat| cat.as_rooted()).collect();
    play_with_cats(children);
    cleanup_after_cats(children, can_gc);
}
```
Tarkka käsitys on, että voimme olla varmoja, ettei GC tapahdu, kun kutsumme `play_with_cats`:ia, mutta GC voi tapahtua, kun kutsumme `cleanup_after_cats`:ia. Syy siihen, miksi jotkin metodit voivat aiheuttaa GC:n ja jotkin eivät, on syvällä SpiderMonkey-yhteyksissä servoon, eikä se yleensä ole ilmeistä.

Mitä tämä tarkoittaa sinulle?
Pohjimmiltaan, jos kutsumasi metodi tarvitsee `CanGc`:n, metodisi tulisi käyttää `CanGc`:ä,
jotta kaikki voivat muistaa olla varovaisia borrow hazard -tilanteiden suhteen koodin ympärillä.


# JSContext, &mut JSContext
Mutta kuten näet, tämä voi olla vaarallista. Entä jos joku unohti käyttää `CanGc`-attribuuttia
funktiokutsussaan, mutta itse asiassa kutsuu GC:tä jossain syvällä kutsupinossa?
Silloin loimme borrow hazard -tilanteen, joka johtaa kaatumisiin, joita emme ehkä ymmärrä.
Ratkaistaksemme tämän otamme käyttöön `JSContext`:in ja `&mut JSContext`:in.
Toisin kuin `CanGc`, nämä tyypit käyttävät Rust-kääntäjän lainaus-sääntöjä ja SpiderMonkey API -wrappeja varmistaakseen, että mikä tahansa API, joka voi käynnistää roskienkeruun, vaatii `&mut JSContext` -argumentin.
Koska nämä arvot ovat uniikkeja, kutsujien täytyy välittää ne.
Vastaavasti mikä tahansa API-funktio, joka _ei voi_ GC:ttä mutta tarvitsee silti pääsyn JS-kontekstiin, ottaa vain `&JSContext` -argumentin.

On myös `NoGc`-tyyppi, joka voidaan konstruoida `JSContext`:ista.
Koska tämä tyyppi lainaa `&mut JSContext`:ia, se tekee mahdottomaksi kutsua mitään koodia, joka vaatii `&mut JSContext` -argumentin (ts. voi käynnistää GC-operaation), kun `NoGc`-arvo on olemassa.

> Huomautus:
> Tällä hetkellä Servo-koodikanta muuttuu aiemmasta `CanGc`-lähestymistavasta
> `JSContext`-lähestymistapaan.
> Saatat nähdä molempia koodikannassa sekä useita escape hatch -ratkaisuja.


# Rootingin suorituskyky

```rust
#[dom_struct]
struct Kittens {
    children: Vec<Cat>
}

impl Kittens {
    fn children(&self) -> Vec<DomRoot<Cat>> {
        self.children.iter().map(|cat| cat.as_rooted()).collect()
    }
}

fn some_function(cats: &Kittens) {
    let happy = cats.children().iter().any(|cat| cat.is_purring());
    if happy {
        println!("Happy cat found!");
    }
}
```

Tämä API kissojen kanssa vuorovaikutukseen on turvallisesti juuritettu, mutta kun lapsia on paljon, juurittaminen ja juuren poisto voivat kasaantua.
Voimme käyttää `JSContext`-tyyppejä varmistaaksemme, ettei rooting-sääntöjä rikkova koodi käynnistä GC-operaatioita, jotka voisivat havaita nämä rikkomukset.

Koska `&mut` on uniikki lainaus, voimme esitellä uuden tyypin:
```rust
struct UnrootedDom<'a, T> {
    inner_cat: Dom<T>,
    js_context: &mut 'a JSContext,
}
```

Sitten aina kun meillä on käsissä `UnrootedDom`, tiedämme, että niin kauan kuin se elää, emme voi kutsua mitään metodeja, joilla on GC.
Seuraava on **virheellistä** koodia.
```rust
#[dom_struct]
struct Kittens {
    children: Vec<UnrootedDom<'_, Cat>>,
}

fn make_cats(cx: &mut JSContext) -> Kittens {}

fn play_with_cat(cx: &JSContext, cat: &Cat) {}

fn cleanup_after_cats(cx: &mut JSContext, cat: &Cat) {}

fn some_function(cx: &mut JSContext) {
    let cats = make_cats(cx);
    for cat in cats.children {
        play_with_cat(cx, cat.inner_cat)
    }
    for cat in cats.children {
        cleanup_after_cats(cx, cat)
    }
}
```

Saamme kääntäjävirheen, että cx on lainattu mutably kerran 'make_cats':issa ja kerran 'cleanup_after_cats':issa.
Mutta huomaa, että kutsu 'play_with_cat':iin on täysin ok.

Tämä on 'UnrootedDomNode':in ja vastaavien 'Unrooted'-metodien ydin. Jotkut näistä voivat myös ottaa 'NoGC'-argumentin
```rust
fn play_with_cats<'a>(no_gc: &'a NoGC, cat: Cat) {}

fn some_function(cx: &mut JSContext, cat: &Cat) {
    play_with_cats(cx.no_gc(), cat);
}
```


## Lisätietoja
TODO:

- <https://github.com/servo/servo/blob/main/components/script/script_thread.rs>
- [JavaScript: Servo's only garbage collector](https://research.mozilla.org/2014/08/26/javascript-servos-only-garbage-collector/)

## SpiderMonkey

Servon SpiderMonkey-integraation nykytila ja näkymät: [https://github.com/gterzian/spidermonkey_servo](https://github.com/servo/servo/wiki/Servo-and-SpiderMonkey-Report)

## Script Thread

- [Microtask queuing](microtasks.md)
