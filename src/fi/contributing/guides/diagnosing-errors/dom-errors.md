# DOM-virheiden diagnosointi

Seuraavan kaltaiset virheviestit kertovat selvästi, että tiettyä DOM-rajapintaa ei ole vielä toteutettu Servossa:
```
[2024-08-16T01:56:15Z ERROR script::dom::bindings::error] Error at https://github.githubassets.com/assets/vendors-node_modules_github_mini-throttle_dist_index_js-node_modules_smoothscroll-polyfill_di-75db2e-686488490524.js:1:9976 AbortSignal is not defined
```

Seuraavan kaltaiset virheviestit eivät kuitenkaan anna paljon ohjausta:
```
[2024-08-16T01:58:25Z ERROR script::dom::bindings::error] Error at https://github.githubassets.com/assets/react-lib-7b7b5264f6c1.js:25:12596 e is undefined
```

Virheviestissä linkitetyn JS-tiedoston avaaminen näyttää usein minifioidun, obfusoidun JS-skriptin, jota on lähes mahdotonta lukea.
Aloitetaan minifioidun koodin purkamisella. Varmista, että `js-beautify`-binääri on polussasi, tai asenna se komennolla:
```
npm install -g js-beautify
```

Aja ongelmasivu Servossa uudelleen sisäänrakennetulla minifiointipalautuksella käytössä:
```
./mach run https://github.com/servo/servo/activity --unminify-js
```

Tämä luo `unminified-js`-hakemiston Servo-repositorion juureen ja tallentaa automaattisesti minifioidut kopiot jokaisesta ulkoisesta JS-skriptistä,
joka haetaan sivun elinkaaren aikana. Servo myös evaluoi skriptien minifioidut versiot, joten rivi- ja sarakenumerot
virheviesteissä muuttuvat:
```
[2024-08-16T02:05:34Z ERROR script::dom::bindings::error] Error at https://github.githubassets.com/assets/react-lib-7b7b5264f6c1.js:3377:66 e is undefined
```

Löydät `react-lib-7b7b5264f6c1.js`-tiedoston hakemistosta `./unminified-js/github.githubassets.com/assets/`, ja jos katsot riviä 3377,
voit alkaa lukea ympäröivää koodia selvittääksesi (toivottavasti), mikä sivulla menee pieleen. Jos koodin tarkastelu ei riitä,
Servo tukee myös minifioidun JS:n _muokkaamista_ paikallisessa välimuistissa!

```
./mach run https://github.com/servo/servo/activity --local-script-source unminified-js
```

Kun `--local-script-source`-argumentti on käytössä, Servo etsii JS-tiedostoja annetusta hakemistosta ensin ennen kuin yrittää hakea
niitä internetistä. Tämä mahdollistaa `console.log(..)`-lauseiden ja muiden hyödyllisten debuggaustekniikoiden lisäämisen ymmärtääksesi,
mitä oikeat verkkosivut havaitsevat. Jos sinun täytyy palata alkuperäiseen sivulähteeseen, aja uudelleen
`--unminify-js`-argumentilla korvataksesi ne uusilla minifioiduilla lähdekooditiedostoilla.
