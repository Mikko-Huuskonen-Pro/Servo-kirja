<!-- TODO: needs copyediting -->

# Microtasks

HTML-spesifikaation mukaan microtask on: "arkikielinen tapa viitata tehtävään, joka luotiin queue a microtask algorithm -algoritmin kautta" ([lähde](https://html.spec.whatwg.org/multipage/#microtask-queue)).
Jokaisella [event-loopilla](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop) — eli window, worker tai worklet — on oma microtask-jononsa.
Jonoon lisätyt tehtävät ajetaan osana [perform a microtask checkpoint](https://html.spec.whatwg.org/multipage/#perform-a-microtask-checkpoint) -algoritmia, jota kutsutaan useista paikoista, pääasiallisesti [tehtävän ajamisen jälkeen](https://html.spec.whatwg.org/multipage/#event-loop-processing-model:perform-a-microtask-checkpoint) tehtäväjonosta, joka ei ole microtask-jono, ja jokainen kutsu tähän algoritmiin tyhjentää microtask-jonon — ajamalla kaikki siihen asti jonoon lisätyt tehtävät ([ilman uudelleensisääntuloisuutta](https://html.spec.whatwg.org/multipage/#performing-a-microtask-checkpoint)).

## Microtask-jono Servossa

[`MicroTaskQueue`](https://github.com/servo/servo/blob/4357751f285c79bf37a8e7a02d4c8dc4f7a8ae69/components/script/microtask.rs#L31) on suoraviivainen spesifikaatioon perustuva toteutus: tehtävälista ja boolean uudelleensisääntuloisuuden estämiseksi checkpointissa.
Yksi [luodaan jokaiselle runtime:lle](https://github.com/servo/servo/blob/4357751f285c79bf37a8e7a02d4c8dc4f7a8ae69/components/script/script_runtime.rs#L519), mikä vastaa spesifikaatiota, koska runtime [luodaan per event-loop](https://github.com/servo/servo/blob/4357751f285c79bf37a8e7a02d4c8dc4f7a8ae69/components/script/script_runtime.rs#L445).
Window event-loopille, joka voi [sisältää useita window-objekteja](https://html.spec.whatwg.org/multipage/#similar-origin-window-agent), jono [jaetaan kaikkien sen sisältämien `GlobalScope`:ien kesken](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/globalscope.rs#L278).
Dedicated workerit [käyttävät child-runtime:a](https://github.com/servo/servo/blob/4357751f285c79bf37a8e7a02d4c8dc4f7a8ae69/components/script/dom/dedicatedworkerglobalscope.rs#L384), mutta sillä on silti oma microtask-jononsa.


### Microtask-jonotus
Tehtävä voidaan jonottaa microtask-jonoon sekä Rustista että JS-moottorista.

- JS:stä: JS-moottori kutsuu [`enqueue_promise_job`](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_runtime.rs#L196) aina, kun sen täytyy jonottaa microtask promise-käsittelijöiden kutsumiseksi.
  Tämä callback-mekanismi asetetaan [kerran per runtime](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_runtime.rs#L520).
  Tämä tarkoittaa, että promisen ratkaiseminen, joko [Rustista](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/promise.rs#L173) tai JS:stä, johtaa tähän callbackiin kutsumiseen ja microtaskin jonotukseen.
  Tiukasti ottaen microtask [jonotetaan silti Rustista](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_runtime.rs#L222).
- Rustista on useita paikkoja, joista microtaskit jonotetaan eksplisiittisesti "natiivista" Rustista:
  - [await a stable state](https://html.spec.whatwg.org/multipage/#await-a-stable-state) -algoritmin toteuttamiseen [script-threadin](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_thread.rs#L966) kautta, ilmeisesti vain script-threadin kautta, mikä tarkoittaa, että worker event-loopit eivät koskaan käytä tätä algoritmia.
  - [dom-queuemicrotask](https://html.spec.whatwg.org/multipage/#dom-queuemicrotask) -algoritmin toteuttamiseen sekä [window](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/window.rs#L928)- että [worker](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/workerglobalscope.rs#L384) event-loopeissa.
  - Ja useissa muissa paikoissa DOM:issa, jotka kaikki voidaan jäljittää [`Microtask`](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/microtask.rs#L39) -enumin variantteihin
  - Microtask voidaan jonottaa vain vaiheista, jotka ajetaan tehtävän sisällä, ei koskaan vaiheista, jotka ajetaan "in-parallel" event-loopin kanssa.

### Microtask Checkpointien ajaminen
[perform-a-microtask-checkpoint](https://html.spec.whatwg.org/multipage/#perform-a-microtask-checkpoint) vastaa [`MicrotaskQueue::checkpoint`](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/microtask.rs#L85) -metodia, ja sitä kutsutaan useissa kohdissa:
- Parserissa, kun [törmätään `script`-tagiin](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/servoparser/mod.rs#L621) tokenisoinnin osana.
  Tämä vastaa [#parsing-main-incdata:perform-a-microtask-checkpoint](https://html.spec.whatwg.org/multipage/#parsing-main-incdata:perform-a-microtask-checkpoint) -kohtaa.
- Uudelleen parserissa, osana [elementin luomista](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/servoparser/mod.rs#L1332).
  Tämä vastaa [#creating-and-inserting-nodes:perform-a-microtask-checkpoint](https://html.spec.whatwg.org/multipage/#creating-and-inserting-nodes:perform-a-microtask-checkpoint) -kohtaa.
- Osana [skriptin ajamisen jälkeistä siivousta](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/bindings/settings_stack.rs#L79).
  Tämä vastaa [#calling-scripts:perform-a-microtask-checkpoint](https://html.spec.whatwg.org/multipage/#calling-scripts:perform-a-microtask-checkpoint) -kohtaa.
- Kahdessa kohdassa ([yksi](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/customelementregistry.rs#L730), [kaksi](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/customelementregistry.rs#L921)) `CustomElementRegistry`:ssä spesifikaation alkuperä näistä kutsuista on epäselvä: ne näyttävät olevan "clean-up after script", mutta metodien dokumentaatiossa viitatuissa spesifikaation osissa ei ole viittauksia tähän.
- [Worker event-loopissa](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/abstractworkerglobalscope.rs#L158), osana [event-loop-processing-model](https://html.spec.whatwg.org/multipage/#event-loop-processing-model) -algoritmin vaihetta 2.8
- Kahdessa paikassa ([yksi](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_thread.rs#L1882), [kaksi](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/script_thread.rs#L1973)) window event-loopissa (`ScriptThread`), jälleen osana [event-loop-processing-model](https://html.spec.whatwg.org/multipage/#event-loop-processing-model) -algoritmin vaihetta 2.8.
  Tämä täytyy yhdistää yhdeksi kutsuksi, ja se, mikä on "tehtävä", täytyy selventää ([TODO(#32003)](https://github.com/servo/servo/issues/32003)).
- [Paint worklet -toteutuksemme](https://github.com/servo/servo/blob/7eac599aa1d6bcf8858c51d90763373f0dd5f289/components/script/dom/paintworkletglobalscope.rs) ei näytä vielä ajavan tätä algoritmia.
