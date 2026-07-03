# Servo-kirja

**[API-dokumentaatio](https://doc.servo.org/servo/)** · **[Englanninkielinen alkuperä](https://book.servo.org)**

***

[_Servo_](https://servo.org) on Rust-ohjelmointikielellä kirjoitettu selainmoottori, jota kehitetään tällä hetkellä seuraaville alustoille:
- 64-bittinen Linux
- 64-bittinen macOS
- 64-bittinen Windows
- Android
- OpenHarmony

Työtä webview-kirjastona käytettäväksi Servo jatkuvat.
Tällä hetkellä suositeltu tapa ajaa Servoa selaimena on [_servoshell_](https://servo.org/download/), [winit](https://crates.io/crates/winit)- ja [egui](https://crates.io/crates/egui)-pohjainen demoseilain.
Jos haluat upottaa Servon omaan sovellukseesi, harkitse [tauri-runtime-verso](https://github.com/versotile-org/tauri-runtime-verso) -ajoaikaa, mukautettua [Tauri](https://tauri.app/)-ajoaikaa, tai [servo-gtk](https://github.com/nacho/servo-gtk) -GTK4-pohjaista selainwidgetiä.

![Kuvakaappaus servoshellistä](../images/servoshell.png)

Tämä kirja opastaa servoshellin kääntämisessä ja ajamisessa, Servon kehittämisessä ja osallistumisessa, Servon arkkitehtuurissa sekä Servon ja sen kirjastojen käytössä.

**Kirja on työn alla!**
Sisällysluettelossa \* merkitsee lukuja, jotka on äskettäin lisätty tai tuotu vanhemmasta dokumentaatiosta ja jotka tarvitsevat vielä oikoluvun tai uudelleentyöstön.

Osallistuminen on aina tervetullutta.
Ehdota muutoksia kunkin sivun oikean yläkulman kynäpainikkeella tai katso lisätietoja reposta [Mikko-Huuskonen-Pro/Servo-kirja](https://github.com/Mikko-Huuskonen-Pro/Servo-kirja).

## Tarvitsetko apua?

Liity [Servo Zulipiin](https://servo.zulipchat.com), jos sinulla on kysyttävää.
Kaikki ovat tervetulleita!
