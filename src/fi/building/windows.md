# Kääntäminen Windowsille

- Lataa [`uv`](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer), [`choco`](https://chocolatey.org/install#individual) ja [`rustup`](https://win.rustup.rs/)
  - Valitse *Quick install via the Visual Studio Community installer*
- Varmista Visual Studio Installerissa, että seuraavat komponentit on asennettu:
  - **Windows 10/11 SDK (anything >= 10.0.19041.0)** (`Microsoft.VisualStudio.Component.Windows{10, 11}SDK.{>=19041}`)
  - **MSVC v143 - VS 2022 C++ x64/x86 build tools (Latest)** (`Microsoft.VisualStudio.Component.VC.Tools.x86.x64`)
  - **C++ ATL for latest v143 build tools (x86 & x64)** (`Microsoft.VisualStudio.Component.VC.ATL`)
- Käynnistä shell uudelleen varmistaaksesi, että `cargo` on käytettävissä
- Asenna muut riippuvuudet: `.\mach bootstrap`
- Käännä servoshell: `.\mach build`

<div class="warning _note">

**Emme suosittele useamman kuin yhden Visual Studio -version asentamista.**
Servo yrittää etsiä oikean Visual Studio -version, mutta vain yhden version asentaminen vähentää virheiden riskiä.
</div>

## Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia eikä ongelmaasi ole listattu alla.


<pre><blockquote><samp><a href="https://github.com/servo/servo/blob/d9f067e998671d16a0274c2a7c8227fec96a4607/python/mach_bootstrap.py#L179">Cannot run mach in a path on a case-sensitive file system on Windows.</a></samp></blockquote></pre>

- Avaa komentokehote tai PowerShell ylläpitäjänä (Win+X, A)
- Poista kirjainkoon herkkyys Servo-repollesi:<br>
  `fsutil file SetCaseSensitiveInfo X:\path\to\servo disable`
  
<pre><blockquote><samp><a href="https://github.com/servo/servo/blob/d86e713a9cb5be2555d63bd477d47d440fa8c832/python/servo/build_commands.py#L460">Could not find DLL dependency: </a>api-ms-win-crt-runtime-l1-1-0.dll</samp></blockquote><blockquote><samp><a href="https://github.com/servo/servo/blob/f76982e2e7f411e2e2fd8e6dbfe92a080acefc54/python/servo/build_commands.py#L531">DLL file `</a>api-ms-win-crt-runtime-l1-1-0.dll<a href="https://github.com/servo/servo/blob/f76982e2e7f411e2e2fd8e6dbfe92a080acefc54/python/servo/build_commands.py#L531">` not found!</a></samp></blockquote></pre>

Etsi polku tiedostoon `Redist\ucrt\DLLs\x64\api-ms-win-crt-runtime-l1-1-0.dll`, esim. `C:\Program Files (x86)\Windows Kits\10\Redist\ucrt\DLLs\x64\api-ms-win-crt-runtime-l1-1-0.dll`.

Aseta sitten `WindowsSdkDir`-ympäristömuuttuja polkuun, joka sisältää `Redist`, esim. `C:\Program Files (x86)\Windows Kits\10`.

<pre><blockquote><samp>thread 'main' panicked at 'Unable to find libclang: "couldn\'t find any valid shared libraries matching: [\'clang.dll\', \'libclang.dll\'], set the `LIBCLANG_PATH` environment variable to a path where one of these files can be found (invalid: [(C:\\Program Files\\LLVM\\bin\\libclang.dll: <strong>invalid DLL (64-bit)</strong>)])"', C:\Users\me\.cargo\registry\src\...</samp></blockquote></pre>

rustup on ehkä asennettu 32-bittisellä oletusisännällä 64-bittisen oletusisännän sijaan, jota Servo tarvitsee.
Tarkista oletusisäntä komennolla `rustup show`, aseta sitten oletusisäntä:

`> rustup set default-host x86_64-pc-windows-msvc`

<pre><blockquote><samp><a href="https://searchfox.org/mozilla-central/rev/058ab60e5020d7c5c98cf82d298aa84626e0cd79/build/moz.configure/util.configure#143-147">ERROR: <strong>GetShortPathName returned a long path name:</strong> `</a>C:/PROGRA~2/Windows Kits/10/<a href="https://searchfox.org/mozilla-central/rev/058ab60e5020d7c5c98cf82d298aa84626e0cd79/build/moz.configure/util.configure#143-147">`. Use `fsutil file setshortname' to create a short name for any components of this path that have spaces.</a></samp></blockquote></pre>

SpiderMonkey (mozjs) vaatii [8.3-tiedostonimet](https://en.wikipedia.org/wiki/8.3_filename) käyttöön Windowsilla ([#26010](https://github.com/servo/servo/issues/26010)).

- Avaa komentokehote tai PowerShell ylläpitäjänä (Win+X, A)
- Ota 8.3-tiedostonimien luonti käyttöön: `fsutil behavior set disable8dot3 0`
- Poista ja asenna uudelleen mikä tahansa, joka sisältää epäonnistuvat polut, kuten Visual Studio tai Windows SDK — tämä on helpompaa kuin 8.3-tiedostonimien lisääminen käsin

<pre><blockquote><samp><a href="https://servo.zulipchat.com/#narrow/channel/263398-general/topic/Build.20issues/near/507644362">= note: lld-link: error: undefined symbol: __std_search_1
>>> referenced by D:\a\mozjs\mozjs\mozjs-sys\mozjs\intl\components\src\NumberFormatterSkeleton.cpp:157</a></samp></blockquote></pre>

Tällaiset ongelmat voivat ilmetä mozjs:n päivityksen yhteydessä, koska päivitys voi riippua uudemmasta MSVC:stä (muista, että vaadimme "Latest" kohdassa [set up your environment](setting-up-your-environment.md#tools-for-windows)!). Ratkaise ongelma käynnistämällä Visual Studio Installer ja asentamalla kaikki saatavilla olevat päivitykset.
