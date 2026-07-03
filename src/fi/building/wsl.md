# Kääntäminen WSL:llä

Servo voidaan kääntää WSL:llä kuten millä tahansa muulla Linux-jakelulla.
Ajaaksesi ei-headless servoshelliä WSL:llä, tarvitset todennäköisesti Windows 10 Build 19044+ tai Windows 11, koska WSL v2 -pääsy tarvitaan graafiseen käyttöliittymään.

1. Ota WSL v2 käyttöön. Katso [Microsoftin ohjeet GUI-sovellusten käyttöön WSL:llä](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps).
2. Seuraa [ohjeita](linux.md) Servon kääntämiseen ja ajamiseen käyttämälläsi WSL-jakelulla (esim. Ubuntu, OpenSuse jne.)

<div class="warning _note">

WSL v2:ssä on vastaavat sovittimet Wayland- ja X11-sovellusten näyttämiseen, vaikka se ei aina toimi suoraan Servon kanssa.
</div>

### Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia eikä ongelmaasi ole listattu alla.

<pre><blockquote><samp>Failed to create event loop</samp></blockquote></pre>

Jos kohtaat välittömän kaatumisen ajon jälkeen, joka viittaa winitiin ja sen alustatoteutukseen, `WAYLAND_DISPLAY=''` pysäyttää kaatumisen.

```
Failed to create events loop: Os(OsError { line: 81, file: "/home/astra/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/winit-0.30.9/src/platform_impl/linux/wayland/event_loop/mod.rs", error: WaylandError(Connection(NoCompositor)) }) (thread main, at ports/servoshell/desktop/cli.rs:34)
   0: servoshell::backtrace::print
             at /home/astra/workspace/servo/ports/servoshell/backtrace.rs:18:5
  ...
  18: main
  19: <unknown>
  20: __libc_start_main
  21: _start
Servo was terminated by signal 11
```

Joko vie muuttuja ympäristöön tai aseta se ennen ajoa:
```shell
export WAYLAND_DISPLAY=''
./mach run

# or

WAYLAND_DISPLAY='' ./mach run

# optionally save the variable long term to your .bashrc profile

echo 'export WAYLAND_DISPLAY=""' >> ~/.bashrc
```

<pre><blockquote><samp>Library libxkbcommon-x11.so could not be loaded</samp></blockquote></pre>

Tämä voi johtua siitä, että jakelusi ei ole asentanut vaadittua kirjastoa. Aja seuraava komento (oletetaan WSL Debian/Ubuntu, säädä muille jakelulle):

```
sudo apt install libxkbcommon-x11-0
```

<pre><blockquote><samp>error: failed to run build command...</samp></blockquote></pre>

Jos kohtaat alla olevan kaltaisen virheen `./mach build` -ajossa WSL:llä, syy voi olla muistin loppuminen (OOM), koska WSL:llä ei ole tarpeeksi RAMia Servon kääntämiseen.
Sinun täytyy kasvattaa WSL:n muistirajaa ja swap-tiedostoa tai päivittää RAM.
```shell
yourusername@PC:~/servo$ ./mach build
No build type specified so assuming `--dev`.
Building `debug` build with crown disabled (no JS garbage collection linting).
   ...
   Compiling script v0.0.1 (/home/yourusername/servo/components/script)

error: failed to run build command...

Caused by:
  process didn't exit successfully: `/home/yourusername/.rustup/toolchains/1.91.0-x86_64-unknown-linux-gnu/bin/rustc --crate-name script --edition=2024 components/script/lib.rs...
  ...
warning: build failed, waiting for other jobs to finish...
```

Luo `C:/user/yourusername/.wslconfig` ja lisää seuraava:
```
[wsl2]
memory=6GB
swap=16GB
swapfile=C:\\Users\\yourusername\\swapfile.vhdx
```

Tallenna tiedosto ja käynnistä WSL uudelleen PowerShellissä:
```
wsl --shutdown
```
