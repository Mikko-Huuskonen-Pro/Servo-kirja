# Servon kääntäminen

Tällä sivulla on tarkempaa tietoa Servon kääntämisestä.
Voit hypätä suoraan ohjeisiin omalle järjestelmällesi:

- [Linux](linux.md)
- [macOS](macos.md)
- [Windows](windows.md)
- [NixOS](nixos.md)
- [WSL](wsl.md)
- [Android](android.md)
- [OpenHarmony](openharmony.md)

# mach

Servon kääntämiseen tarvitset `mach`-työkalun.
`mach` on Python-ohjelma, joka helpottaa Servon parissa työskentelyä monin tavoin: kääntäminen ja ajaminen, testien ajo ja riippuvuuksien päivitys.

**Windows-käyttäjät:** sinun täytyy korvata `./mach` komennoissa merkkijonolla `.\mach`, jos käytät cmd:tä.

Käytä `--help` listataksesi alikomennot tai saadaksesi apua tiettyyn alikomentoon:

```sh
$ ./mach --help
$ ./mach build --help
```

Kun käytät machia ajamaan toista ohjelmaa, kuten servoshelliä, sillä ohjelmalla voi olla omia valintoja samoilla nimillä kuin machilla.
Voit käyttää `--` (välilyönneillä ympäröitynä) kertoaksesi machille, ettei se koske myöhempiin valintoihin vaan jättää ne toiselle ohjelmalle.

```sh
$ ./mach run --help         # Gets help for `mach run`.
$ ./mach run -d --help      # Still gets help for `mach run`.
$ ./mach run -d -- --help   # Gets help for the debug build of servoshell.
```

Tämä pätee myös Servon yksikkötesteihin, joissa on kolme valintakerrosta: mach-valinnat, `cargo test` -valinnat ja [libtest-valinnat](https://doc.rust-lang.org/cargo/commands/cargo-test.html#description).

```sh
$ ./mach test-unit --help           # Gets help for `mach test-unit`.
$ ./mach test-unit -- --help        # Gets help for `cargo test`.
$ ./mach test-unit -- -- --help     # Gets help for the test harness (libtest).
```

Työtä jatketaan, jotta Servo voidaan kääntää ilman machia.
Harkitse aina, voitko käyttää natiivia Cargo-toiminnallisuutta ennen kuin lisäät uutta toiminnallisuutta machiin.

## Käännösprofiilit

On kolme pääkäännösprofiilia, joita voit kääntää ja käyttää toisistaan riippumatta:

- debug-käännökset, joiden avulla voit käyttää debuggeria (lldb) (oletus, jos profiilia ei anneta)
- release-käännökset, jotka kääntyvät hitaammin mutta ovat suorituskykyisempiä
- production-käännökset, joita käytetään vain virallisissa julkaisuissa

<table>
<thead>
    <tr>
        <th></th>
        <th>debug</th>
        <th>release</th>
        <th>production</th>
    </tr>
</thead>
<tbody>
    <tr>
        <th>mach-valinta</th>
        <td><code>-d<br>--debug</code></td>
        <td><code>-r<br>--release</code></td>
        <td><code>--prod<br>--production</code></td>
    </tr>
    <tr>
        <th>optimoitu?</th>
        <td><a href="https://doc.rust-lang.org/cargo/reference/profiles.html#dev">ei</a></td>
        <td><a href="https://github.com/servo/servo/blob/457d37d94ee6966cad377c373d333a00c637e1ae/Cargo.toml#L153">kyllä</a></td>
        <td>kyllä, <a href="https://github.com/servo/servo/blob/9457a40ca2cd4b9530ba7c5334c82f3b3f2e7ac8/Cargo.toml#L177-L182">enemmän kuin</a> <strong>release</strong></td>
    </tr>
    <tr>
        <th>suurin RUST_LOG-taso</th>
        <td><code>trace</code></td>
        <td><code>info</code></td>
        <td><code>info</code></td>
    </tr>
    <tr>
        <th>debug-väitteet?</th>
        <td>kyllä</td><td>kyllä(!)</td><td>ei</td>
    </tr>
    <tr>
        <th>debug-tiedot?</th>
        <td>kyllä</td><td>ei</td><td>ei</td>
    </tr>
    <tr>
        <th>symbolit?</th>
        <td>kyllä</td><td>ei</td><td>kyllä</td>
    </tr>
    <tr>
        <th>etsii resursseja<br>nykyisestä työhakemistosta?</th>
        <td>kyllä</td><td>kyllä</td><td>ei(!)</td>
    </tr>
</tbody>
</table>

Lisäksi on kaksi erityistä production-käännöksen varianttia suorituskykyyn liittyviin käyttötarkoituksiin:

- `production-stripped` -käännökset sopivat Servon benchmarkkaukseen ajan myötä, kun debug-symbolit on poistettu nopeampaa käynnistystä varten
- `profiling` -käännökset sopivat [profilointiin](../contributing/profiling.md) ja suorituskykyongelmien vianmääritykseen; ne käyttäytyvät kuin debug- tai release-käännös, mutta niiden suorituskyky on sama kuin production-käännöksellä

<table>
<thead>
    <tr>
        <th></th>
        <th>production</th>
        <th>production-stripped</th>
        <th>profiling</th>
    </tr>
</thead>
<tbody>
    <tr>
        <th>mach <code>--profile</code></th>
        <td><code>production</code></td>
        <td><code>production-stripped</code></td>
        <td><code>profiling</code></td>
    </tr>
    <tr>
        <th>debug-tiedot?</th>
        <td>ei</td><td>ei</td><td>kyllä</td>
    </tr>
    <tr>
        <th>symbolit?</th>
        <td>kyllä</td><td>ei</td><td>kyllä</td>
    </tr>
    <tr>
        <th>etsii resursseja<br>nykyisestä työhakemistosta?</th>
        <td>ei</td><td>ei</td><td>kyllä(!)</td>
    </tr>
</tbody>
</table>

Voit muuttaa näitä asetuksia servobuild-tiedostossa (katso [servobuild.example](https://github.com/servo/servo/blob/b79e2a0b6575364de01b1f89021aba0ec3fcf399/servobuild.example)) tai juuren [Cargo.toml](https://github.com/servo/servo/blob/b79e2a0b6575364de01b1f89021aba0ec3fcf399/Cargo.toml) -tiedostossa.

## Valinnaiset käännösasetukset

Joitakin käännösasetuksia voi ottaa käyttöön vain manuaalisesti:

- **AddressSanitizer-käännökset** otetaan käyttöön komennolla `./mach build --with-asan`
- **ThreadSanitizer-käännökset** otetaan käyttöön komennolla `./mach build --with-tsan`
- **crown-linttaus** on suositeltava DOM-koodia muokattaessa, ja se otetaan käyttöön komennolla `./mach build --use-crown`
- **SpiderMonkey debug -käännökset** otetaan käyttöön komennolla `./mach build --debug-mozjs` tai `[build] debug-mozjs = true` servobuild-tiedostossasi

Täydellisen argumenttilistan näet ajamalla `./mach build --help`.

## Servoshellin ajaminen

Kun käännät itse, servoshell on polussa `target/debug/servo` tai `target/release/servo`.
Voit ajaa sen suoraan kuten yllä, mutta suosittelemme käyttämään [machia](#mach) sen sijaan.

Ajaaksesi servoshellin machilla, korvaa `./servo` komennoilla `./mach run -d --` tai `./mach run -r --` riippuen [käännösprofiilista](#käännösprofiilit), jonka haluat ajaa.
Esimerkiksi molemmat alla olevat komennot ajavat debug-käännöksen servoshelliä samoilla valinnoilla:

```sh
$ target/debug/servo https://demo.servo.org
$ ./mach run -d -- https://demo.servo.org
```
