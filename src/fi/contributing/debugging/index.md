# Debuggaus

Yksi yksinkertaisimmista tavoista debugata Servoa on tulostaa kiinnostavia muuttujia [`println!`](https://doc.rust-lang.org/std/macro.println.html)-, [`eprintln!`](https://doc.rust-lang.org/std/macro.eprintln.html)- tai [`dbg!`](https://doc.rust-lang.org/std/macro.dbg.html)-makroilla.
Yleensä niitä tulisi käyttää vain tilapäisesti; sinun täytyy poistaa ne tai muuntaa ne kunnolliseksi debug-lokituksesi ennen kuin pull request yhdistetään.

## Debug-lokitus `log`-kirjastolla ja `RUST_LOG`-muuttujalla

Servo käyttää [`log`](https://crates.io/crates/log)-kirjastoa pitkäaikaiseen debug-lokitukseen ja virheviesteihin:

```rust
log::error!("hello");
log::warn!("hello");
log::info!("hello");
log::debug!("hello");
log::trace!("hello");
```

Toisin kuin `println!`-kaltaiset makrot, `log` lisää aikaleiman ja kertoo, mistä viesti tuli:

```
[2024-05-01T09:07:42Z ERROR servoshell::app] hello
[2024-05-01T09:07:42Z WARN  servoshell::app] hello
[2024-05-01T09:07:42Z INFO  servoshell::app] hello
[2024-05-01T09:07:42Z DEBUG servoshell::app] hello
[2024-05-01T09:07:42Z TRACE servoshell::app] hello
```

Voit suodattaa `log`-kirjaston tulostetta `RUST_LOG`-muuttujalla tason (`off`, `error`, `warn`, `info`, `debug`, `trace`) ja/tai viestin lähteen, eli niin sanotun „targetin”, mukaan.
Yleensä target on Rust-moduulin polku kuten `servoshell::app`, mutta on myös erityisiä targeteja (ks. [§ *Tapahtumien jäljittäminen*](#tapahtumien-jaljittaminen)).
Aseta `RUST_LOG` lisäämällä se komennon eteen tai käyttämällä `export`-komentoa:

```sh
$ RUST_LOG=warn ./mach run -d test.html     # Uses the prepended RUST_LOG.
$ export RUST_LOG=warn
$ ./mach run -d test.html                   # Uses the exported RUST_LOG.
```

Katso [`env_logger`-dokumentaatiosta](https://docs.rs/env_logger/0.11.3/env_logger/index.html#enabling-logging) lisätietoja, mutta tässä muutamia esimerkkejä:

- ota käyttöön kaikki viestit `debug`-tasolle asti, mutta ei `trace`-tasoa:
  <br>`RUST_LOG=debug`
- ota käyttöön kaikki viestit kohteista `servo::*`, `servoshell::*` tai mistä tahansa targetista, joka alkaa `servo`:
  <br>`RUST_LOG=servo=trace` (tai pelkkä `RUST_LOG=servo`)
- ota käyttöön kaikki viestit kohteista, jotka alkavat `style`, mutta vain `error`- ja `warn`-viestit kohteesta `style::rule_tree`:
  <br>`RUST_LOG=style,style::rule_tree=warn`

Huomaa, että vaikka lokiviesti suodatetaan pois, se voi silti vaikuttaa suorituskykyyn, vaikkakin vain hieman.
[Eräät käännökset](../../building/building.md#build-profiles) Servosta, **mukaan lukien viralliset yöjulkaisut**, poistavat DEBUG- ja TRACE-viestit käännösaikana, joten niiden ottaminen käyttöön `RUST_LOG`-muuttujalla ei vaikuta mihinkään.

### Tapahtumien jäljittäminen

**Constellationissa**, **compositorissa** ja **servoshellissä** lokataan muihin komponentteihin lähetetyt ja niistä vastaanotetut viestit targeteilla muodossa `component>other@Event` tai `component<other@Event`.
Tämä tarkoittaa, että voit valita ajonaikaisesti, mitä tapahtumatyyppejä lokataan `RUST_LOG`-muuttujalla!

Esimerkiksi **constellationissa** ([lisätietoja](https://github.com/servo/servo/blob/bccbc87db7b986cae31c8f14f0a130336f8417b2/components/constellation/tracing.rs)):

- jäljitä vain scriptin tapahtumat:
  <br>`RUST_LOG='constellation<=off,constellation<script@'`
- jäljitä kaikki tapahtumat paitsi ReadyToPresent-tapahtumat:
  <br>`RUST_LOG='constellation<,constellation<compositor@ReadyToPresent=off'`
- jäljitä vain scriptin InitiateNavigateRequest-tapahtumat:
  <br>`RUST_LOG='constellation<=off,constellation<script@InitiateNavigateRequest'`

**Compositorissa** ([lisätietoja](https://github.com/servo/servo/blob/bccbc87db7b986cae31c8f14f0a130336f8417b2/components/compositing/tracing.rs)):

- jäljitä vain MoveResizeWebView-tapahtumat:
  <br>`RUST_LOG='compositor<constellation@MoveResizeWebView'`
- jäljitä kaikki tapahtumat paitsi Forwarded-tapahtumat:
  <br>`RUST_LOG=compositor<,compositor<constellation@Forwarded=off`

**Servoshellissä** ([lisätietoja](https://github.com/servo/servo/blob/01a9b317d4a6710547b8b0c0c476cc3b82251044/ports/servoshell/desktop/tracing.rs)):

- jäljitä vain servon tapahtumat:
  <br>`RUST_LOG='servoshell<=off,servoshell>=off,servoshell<servo@'`
- jäljitä kaikki tapahtumat paitsi AxisMotion-tapahtumat:
  <br>`RUST_LOG='servoshell<,servoshell>,servoshell<winit@WindowEvent(AxisMotion)=off'`
- jäljitä vain winitin ikkunan siirto -tapahtumat:
  <br>`RUST_LOG='servoshell<=off,servoshell>=off,servoshell<winit@WindowEvent(Moved)'`

Tapahtumien jäljittäminen voi tuottaa valtavan määrän tulostetta.
Yleensä suosittelemme seuraavaa konfiguraatiota, jotta tuloste pysyy käyttökelpoisena:

- `constellation<,constellation>,constellation<compositor@ForwardEvent(MouseMoveEvent)=off,constellation<compositor@LogEntry=off,constellation<compositor@ReadyToPresent=off,constellation<script@LogEntry=off`
- `compositor<,compositor>`
- `servoshell<,servoshell>,servoshell<winit@DeviceEvent=off,servoshell<winit@MainEventsCleared=off,servoshell<winit@NewEvents(WaitCancelled)=off,servoshell<winit@RedrawEventsCleared=off,servoshell<winit@RedrawRequested=off,servoshell<winit@UserEvent(WakerEvent)=off,servoshell<winit@WindowEvent(CursorMoved)=off,servoshell<winit@WindowEvent(AxisMotion)=off`

## Muu debug-lokitus

`mach run` tekee tämän automaattisesti, mutta tulosta backtrace, kun Servo panikoi:

```sh
$ RUST_BACKTRACE=1 target/debug/servo test.html
```

Käytä `-Z` (`-- --debug`) ottaaksesi debug-asetukset käyttöön.
Esimerkiksi tulosta stacking context -puu jokaisen layoutin jälkeen tai hae apua näihin asetuksiin:

```sh
$ ./mach run -Z stacking-context-tree test.html
$ ./mach run -Z help            # Lists available debug options.
$ ./mach run -- --debug help    # Same as above: lists available debug options.
$ ./mach run --debug            # Not the same! This just chooses target/debug.
```

Vaihtoehtoisesti voit käyttää `SERVO_DIAGNOSTICS`-ympäristömuuttujaa diagnostiikka-asetusten määrittämiseen.

```sh
$ SERVO_DIAGNOSTICS=style-tree ./mach run test.html
$ SERVO_DIAGNOSTICS=help ./mach run    # Lists available debug options.
$ export SERVO_DIAGNOSTICS=style-tree,display-list
$ ./mach run test.html            # Uses the exported SERVO_DIAGNOSTICS.
```

`SERVO_DIAGNOSTICS`-ympäristömuuttuja hyväksyy pilkuilla erotetut diagnostiikka-asetukset, samat kuin `-Z`-valinnan kautta saatavilla olevat.
Tämä ominaisuus on saatavilla vain debug- ja release-käännöksissä, ei production-käännöksissä.

macOS:llä voit myös lisätä Cocoa-spesifisiä debug-asetuksia ylimääräisen `--`-merkitsimen jälkeen:

```sh
$ ./mach run -- test.html -- -NSShowAllViews YES
```

## Servoshellin ajaminen debuggerilla

Aja servoshell debuggerilla käyttämällä `--debugger-cmd`-valintaa.
Huomaa, että jos valitset `gdb` tai `lldb`, käytämme automaattisesti `rust-gdb`:tä ja `rust-lldb`:tä.

```sh
$ ./mach run --debugger-cmd=gdb test.html   # Same as `--debugger-cmd=rust-gdb`.
$ ./mach run --debugger-cmd=lldb test.html  # Same as `--debugger-cmd=rust-lldb`.
```

Välittääksesi debuggerille lisäasetuksia sinun täytyy ajaa debugger itse:

```sh
$ ./mach run --debugger-cmd=gdb -ex=r test.html         # Passes `-ex=r` to servoshell.
$ rust-gdb -ex=r --args target/debug/servo test.html    # Passes `-ex=r` to gdb.

$ ./mach run --debugger-cmd=lldb -o r test.html         # Passes `-o r` to servoshell.
$ rust-lldb -o r -- target/debug/servo test.html        # Passes `-o r` to lldb.

$ ./mach run --debugger-cmd=rr -M test.html             # Passes `-M` to servoshell.
$ rr record -M target/debug/servo test.html             # Passes `-M` to rr.
```

Monet debuggerit tarvitsevat lisäasetuksia erottamaan servoshellin argumentit omista valinnoistaan, ja `--debugger-cmd` välittää nämä asetukset automaattisesti [muutamille debuggereille](https://github.com/servo/servo/blob/bccbc87db7b986cae31c8f14f0a130336f8417b2/third_party/mozdebug/mozdebug/mozdebug.py#L32-L48), mukaan lukien `gdb` ja `lldb`.
Muille debuggereille `--debugger-cmd` toimii vain, jos debugger ei tarvitse ylimääräisiä asetuksia:

```sh
$ ./mach run --debugger-cmd=rr test.html                    # Good, because it’s...
#  servoshell arguments        ^^^^^^^^^
$ rr target/debug/servo test.html                           # equivalent to this.
#  servoshell arguments ^^^^^^^^^

$ ./mach run --debugger-cmd=renderdoccmd capture test.html  # Bad, because it’s...
#                renderdoccmd arguments? ^^^^^^^
#                  servoshell arguments          ^^^^^^^^^
$ renderdoccmd target/debug/servo capture test.html         # equivalent to this.
# => target/debug/servo is not a valid command.

$ renderdoccmd capture target/debug/servo test.html         # Good.
#              ^^^^^^^ renderdoccmd arguments
#                    servoshell arguments ^^^^^^^^^
```

## Debuggaus gdb:llä tai lldb:llä

Etsi funktio nimen tai regexin perusteella:

```
(lldb) image lookup -r -n <name>
(gdb) info functions <name>
```

Listaa käynnissä olevat säikeet:

```
(lldb) thread list
(lldb) info threads
```

Muita gdb- tai lldb-komentoja:

```
(gdb) b a_servo_function    # Add a breakpoint.
(gdb) run                   # Run until breakpoint is reached.
(gdb) bt                    # Print backtrace.
(gdb) frame n               # Choose the stack frame by its number in `bt`.
(gdb) next                  # Run one line of code, stepping over function calls.
(gdb) step                  # Run one line of code, stepping into function calls.
(gdb) print varname         # Print a variable in the current scope.
```

Katso [tätä gdb-oppasta](http://www.unknownroad.com/rtfm/gdbtut/gdbtoc.html) tai [tätä lldb-oppasta](https://lldb.llvm.org/use/tutorial.html) lisätietoja varten.

Voit tarkastella muuttujia lldb:ssä myös kirjoittamalla `gui` ja käyttämällä nuolinäppäimiä muuttujien laajentamiseen:

```
(lldb) gui
┌──<Variables>───────────────────────────────────────────────────────────────────────────┐
│ ◆─(&mut gfx::paint_task::PaintTask<Box<CompositorProxy>>) self = 0x000070000163a5b0    │
│ ├─◆─(msg::constellation_msg::PipelineId) id                                            │
│ ├─◆─(url::Url) _url                                                                    │
│ │ ├─◆─(collections::string::String) scheme                                             │
│ │ │ └─◆─(collections::vec::Vec<u8>) vec                                                │
│ │ ├─◆─(url::SchemeData) scheme_data                                                    │
│ │ ├─◆─(core::option::Option<collections::string::String>) query                        │
│ │ └─◆─(core::option::Option<collections::string::String>) fragment                     │
│ ├─◆─(std::sync::mpsc::Receiver<gfx::paint_task::LayoutToPaintMsg>) layout_to_paint_port│
│ ├─◆─(std::sync::mpsc::Receiver<gfx::paint_task::ChromeToPaintMsg>) chrome_to_paint_port│
└────────────────────────────────────────────────────────────────────────────────────────┘
```

Jos lldb kaatuu tietyillä riveillä, joissa on `profile()`-funktio, et ole ainoa.
Kommentoi profilointikoodi pois ja pidä vain sisäinen funktio — sen pitäisi riittää.

## Kääntävä debuggaus rr:llä (vain Linux)

[rr](https://rr-project.org) on kuin gdb, mutta sen avulla voit kelata taaksepäin.
Aloita ajamalla servoshell rr:n kautta:

```sh
$ ./mach run --debugger=rr test.html    # Either this...
$ rr target/debug/servo test.html       # ...or this.
```

Toista sitten jälki ja käytä gdb- tai rr-komentoja:

```
$ rr replay
(rr) continue
(rr) reverse-cont
```

Aja yksi tai useampi testi toistuvasti, kunnes tulos on odottamaton:

```sh
$ ./mach test-wpt --chaos path/to/test [path/to/test ...]
```

rr:n tallentamat jäljet voivat viedä paljon tilaa.
Poista ne hakemistosta `~/.local/share/rr`.

## OpenGL-debuggaus RenderDocilla (vain Linux tai Windows)

[RenderDoc](https://renderdoc.org/docs/) mahdollistaa Servon OpenGL-toiminnan debuggauksen.
Aloita ajamalla servoshell renderdoccmd:n kautta:

```sh
$ renderdoccmd capture -d . target/debug/servo test.html
```

Kun servoshell on käynnissä, aja `qrenderdoc` ja valitse **File** > **Attach to Running Instance**.
Kun yhteys on muodostettu, voit painaa **F12** tai **Print Screen** kaapataksesi ruudun.
