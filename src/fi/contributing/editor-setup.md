# Editorin asetukset

On erittäin suositeltavaa asettaa editorisi tukemaan [`rust-analyzer`](https://rust-analyzer.github.io/)-työkalua.
Vaikka se vaatii kohtuullisen määrän RAM-muistia ja CPU:ta, se parantaa Servon parissa työskentelyä merkittävästi, koska editorisi voi tarjota koodin täydentämistä, reaaliaikaisia käännösvirheitä ja varoituksia sekä helposti saatavilla olevan [rustdoc](https://doc.rust-lang.org/rustdoc/index.html)-dokumentaation.
Valitettavasti `rust-analyzer` yrittää ajaa `cargo`-komentoa ilman `mach`-komentoa, mikä aiheuttaa ongelmia Servon erityisen käännöskonfiguraation vuoksi.
Käytä seuraavia ohjeita IDE:n konfigurointiin.
Lisäykset ovat tervetulleita!

## Visual Studio Code

On suositeltavaa lisätä seuraava projektikohtaisiin asetuksiin tiedostoon `.vscode/settings.json`:

```json
{
    "rust-analyzer.rustfmt.overrideCommand": [ "./mach", "fmt" ],
    "rust-analyzer.check.overrideCommand": [
        "./mach",
        "clippy",
        "--message-format=json",
        "--target-dir",
        "target/lsp",
        "--features",
        "tracing,tracing-perfetto"
    ],
    "rust-analyzer.cargo.buildScripts.overrideCommand": [
        "./mach",
        "clippy",
        "--message-format=json",
        "--target-dir",
        "target/lsp",
        "--features",
        "tracing,tracing-perfetto"    ],
}
```

Huomioita:

- Yllä olevassa katkelmassa kielipalvelin kääntää omaan target-hakemistoonsa `target/lsp`, jotta vältetään ei-toivotut uudelleenkäännökset.
  Jos haluat säästää levytilaa, voit poistaa `--target-dir`- ja `target/lsp`-argumentit, jolloin käytetään oletustarget-hakemistoa.
- Ota [valinnaiset käännösasetukset](../building/building.md#optional-build-settings) käyttöön lisäämällä ne konfiguraatiotiedoston build-argumenttilistaan.
- **Windows:** Windowsissa sinun on käytettävä `./mach.bat`-komentoa `./mach`-komennon sijaan.
  Muuten saatat saada virheen, että suoritettu komento ei ole kelvollinen Win32-sovellus.
- **Ristikäännös:** Jos ristikäännät, voit ohittaa cargo-käytettävän targetin lisäämällä `"rust-analyzer.cargo.target": "aarch64-linux-android"`.
  Huomaa, että jotkin LSP-ominaisuudet eivät välttämättä toimi tässä konfiguraatiossa.
## Zed

Jos käytät Zedia, sinun on tehtävä hyvin samanlaista kuin Visual Studio Code -kohdassa kuvataan, mutta Zed-konfiguraatiotiedosto odottaa hieman erilaista syntaksia.
Tiedostossa `./zed/settings.json` tarvitset jotain tällaista:

```json
{
  "lsp": {
    "rust-analyzer": {
      "initialization_options": {
        "checkOnSave": true,
        "check": {
          "overrideCommand": [
            "./mach",
            "clippy",
            "--message-format=json",
            "--target-dir",
            "target/lsp",
            "--feature",
            "tracing,tracing-perfetto"
          ]
        },
        "cargo": {
          "allTargets": false,
          "buildScripts": {
            "overrideCommand": [
              "./mach",
              "clippy",
              "--message-format=json",
              "--target-dir",
              "target/lsp",
              "--feature",
              "tracing,tracing-perfetto"
            ]
          }
        },
        "rustfmt": {
          "extraArgs": [
            "--config",
            "unstable_features=true",
            "--config",
            "binop_separator=Back",
            "--config",
            "imports_granularity=Module",
            "--config",
            "group_imports=StdExternalCrate"
          ]
        }
      }
    }
  }
}
```

### Python

Servo sisältää Python-skriptejä osana build-työkaluja ja tiettyjä testityyppejä.
Zedin oletus-Python-konfiguraatio ei välttämättä tarjoa kunnollista staattista koodianalyysia.

Servo on konfiguroitu toimimaan Pyrefly-kielipalvelimen kanssa, jonka voit ottaa käyttöön alla olevan kaltaisella konfiguraatiolla.
Tarvitset myös [Pyrefly](https://zed.dev/extensions/pyrefly) Zed-laajennuksen.

```json
{
  "lsp": {
    "pyrefly": {
      "binary": {
        "path": ".venv/bin/pyrefly",
        "arguments": ["lsp"],
      },
      "settings": {
        "python": {
          "pythonPath": ".venv/bin/python",
        },
        "pyrefly": {
          "python_interpreter": ".venv/bin/python",
        },
      },
    },
  },
  "languages": {
    "Python": {
      "language_servers": ["pyrefly", "!pyright", "!pylsp"],
    },
  },
}
```

### NixOS

NixOS:ssa saatat saada virheitä `pkg-config`- tai `crown`-paketeista:

> thread 'main' panicked at 'called \`Result::unwrap()\` on an \`Err\` value: "Could not run \`PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=\\"1\\" PKG_CONFIG_ALLOW_SYSTEM_LIBS=\\"1\\" \\"pkg-config\\" \\"--libs\\" \\"--cflags\\" \\"fontconfig\\"\`

> [ERROR rust_analyzer::main_loop] FetchWorkspaceError: rust-analyzer failed to load workspace: Failed to load the project at /path/to/servo/Cargo.toml: Failed to read Cargo metadata from Cargo.toml file /path/to/servo/Cargo.toml, Some(Version { major: 1, minor: 74, patch: 1 }): Failed to run \`cd "/path/to/servo" && "cargo" "metadata" "--format-version" "1" "--manifest-path" "/path/to/servo/Cargo.toml" "--filter-platform" "x86_64-unknown-linux-gnu"\`: \`cargo metadata\` exited with an error: error: could not execute process \`crown -vV\` (never executed)

`mach` välittää eri RUSTFLAGS-arvot Rust-kääntäjälle kuin pelkkä `cargo`, joten jos yrität kääntää Servon `cargo`-komennolla, se kumoaa `mach`-komennon työn (ja päinvastoin).
Tämän vuoksi, ja koska Servo voidaan tällä hetkellä kääntää vain `mach`-komennolla, sinun on konfiguroitava rust-analyzer-laajennus käyttämään `mach`-komentoa tiedostossa `.vscode/settings.json`:

#### `crown`-käyttö

Jos käytät `--use-crown`-valitsinta, sinun pitäisi myös asettaa CARGO_BUILD_RUSTC tiedostossa `.vscode/settings.json` seuraavasti, jossa `/nix/store/.../crown` on komennon `nix-shell --run 'command -v crown'` tulos.

```
{
    "rust-analyzer.server.extraEnv": {
        "CARGO_BUILD_RUSTC": "/nix/store/.../crown",
    },
}
```

Näiden asetusten pitäisi riittää siihen, ettei sinun tarvitse ajaa `code .` `nix-shell`-ympäristöstä, mutta voit kokeilla sitä, jos ongelmia on yhä.

#### Ongelmia proc-makrojen kanssa

Kun otat rust-analyzerin proc-makrotuen käyttöön, saatat alkaa nähdä virheitä kuten

> proc macro \`MallocSizeOf\` not expanded: Cannot create expander for /path/to/servo/target/debug/deps/libfoo-0781e5a02b945749.so: unsupported ABI \`rustc 1.69.0-nightly (dc1d9d50f 2023-01-31)\` rust-analyzer(unresolved-proc-macro)

Tämä tarkoittaa, että rust-analyzer käyttää väärää proc-makropalvelinta, ja sinun on konfiguroitava oikea manuaalisesti.
Käytä mach-komentoa kysyäksesi nykyisen sysroot-polun ja kopioi tulosteen viimeinen rivi:

```
$ ./mach rustc --print sysroot
NOTE: Entering nix-shell /path/to/servo/shell.nix
info: component 'llvm-tools' for target 'x86_64-unknown-linux-gnu' is up to date
/home/me/.rustup/toolchains/nightly-2023-02-01-x86_64-unknown-linux-gnu
```

Konfiguroi sitten joko sysroot-polku tai proc-makropalvelimen polku tiedostossa `.vscode/settings.json`:

```
{
    "rust-analyzer.procMacro.enable": true,
    "rust-analyzer.cargo.sysroot": "[paste what you copied]",
    "rust-analyzer.procMacro.server": "[paste what you copied]/libexec/rust-analyzer-proc-macro-srv",
}
```

## Emacs

Emacsissa on kaksi LSP-asiakasimplementaatiota: [eglot](https://elpa.gnu.org/packages/eglot.html), joka on Emacsin sisäänrakennettu paketti, ja [emacs-lsp](https://emacs-lsp.github.io/lsp-mode/).

### Eglot

Komentojen ohittamiseksi meidän on asetettava [`eglot-workspace-configuration`](https://elpa.gnu.org/packages/doc/eglot.html#Project_002dspecific-configuration). Luo tätä varten `.dir-locals.el`-tiedosto Servo-checkoutisi ylätason hakemistoon seuraavalla sisällöllä:


```elisp
;;; Directory Local Variables         -*- no-byte-compile: t; -*-
;;; For more information see (info "(emacs) Directory Variables")

((nil . ((eglot-workspace-configuration
          . (:rust-analyzer
             (:rustfmt (:overrideCommand ["./mach" "fmt"])
                       :check (:overrideCommand
                               ["./mach" "clippy" "--message-format=json" "--target-dir" "target/lsp" "--features" "tracing,tracing-perfetto"])
                       :cargo (:buildScripts
                               (:overrideCommand
                                ["./mach" "clippy" "--message-format=json" "--target-dir" "target/lsp" "--features" "tracing,tracing-perfetto"]))))))))

```

Kun olet käynnistänyt eglotin (`M-x eglot`), voit tarkistaa käytössä olevan workspace-konfiguraation komennolla `M-x eglot-show-workspace-configuration`.
