# Kääntäminen Linuxille

- Asenna `curl`:
  - Arch: `sudo pacman -S --needed curl`
  - Debian, Ubuntu: `sudo apt install curl`
  - Fedora: `sudo dnf install curl`
  - Gentoo: `sudo emerge net-misc/curl`
- Asenna `uv`: `curl -LsSf https://astral.sh/uv/install.sh | sh` 
- Asenna `rustup`: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- Käynnistä shell uudelleen varmistaaksesi, että `cargo` on käytettävissä
- Asenna muut riippuvuudet: `./mach bootstrap`
- Käännä servoshell: `./mach build`

## Tukemattomat jakelut 

Jos `./mach boostrap` ilmoittaa, että jakelusi ei ole tuettu, sinun täytyy asentaa riippuvuudet manuaalisesti.
Alla on ohjeita käännösriippuvuuksien asentamiseen erilaisille jakelutyypeille.
Jos jakelusi ei ole listalla, suositellaan sovittamaan lista järjestelmäsi pakettinimille.
Päivitykset tähän listaan ovat erittäin tervetulleita!

### Arch ja Manjaro { #dependencies-for-arch }

<!-- https://archlinux.org/packages/ -->
- `sudo pacman -S --needed curl`

- `sudo pacman -S --needed base-devel git mesa cmake libxmu pkg-config ttf-fira-sans harfbuzz ccache llvm clang autoconf2.13 gstreamer gstreamer-vaapi gst-plugins-base gst-plugins-good gst-plugins-bad gst-plugins-ugly vulkan-icd-loader wireshark-cli`

### Debian-tyyppiset{ #dependencies-for-debian }
**(mukaan lukien elementary OS, KDE neon, Linux Mint, Pop!_OS, Raspbian, TUXEDO OS, Ubuntu)**

<!-- https://packages.debian.org -->
<!-- https://packages.ubuntu.com -->
- `sudo apt install curl`

<!-- see python/servo/platform/building/linux_packages/apt/ in servo for how to update this -->
<!-- be sure to *remove* `libgstreamer-plugins-good1.0-dev` from this list, due to the note below -->
- `sudo apt install {{#include linux_packages/apt_common.txt}}`

**Huom:** Ubuntu-pohjaisissa jakelussa varmista, että sisällytät myös paketin `libgstreamer-plugins-good1.0-dev` yllä lueteltujen pakettien lisäksi.

### Fedora-tyyppiset { #dependencies-for-fedora }

<!-- Update python/servo/platform/linux_packages/dnf in servo as changes will sync to the book -->
* `sudo dnf install {{#include linux_packages/dnf_base.txt}}`

### Gentoo-tyyppiset { #dependencies-for-gentoo }

<!-- https://packages.gentoo.org -->
- `sudo emerge net-misc/curl media-libs/freetype media-libs/mesa dev-util/gperf dev-libs/openssl media-libs/harfbuzz dev-util/ccache sys-libs/libunwind x11-libs/libXmu x11-base/xorg-server sys-devel/clang media-libs/gstreamer media-libs/gst-plugins-base media-libs/gst-plugins-good media-libs/gst-plugins-bad media-libs/gst-plugins-ugly media-libs/vulkan-loader`

### openSUSE { #dependencies-for-opensuse }

<!-- https://search.opensuse.org/packages/ -->
- `sudo zypper install libX11-devel libexpat-devel Mesa-libEGL-devel Mesa-libGL-devel cabextract cmake dbus-1-devel fontconfig-devel freetype-devel gcc-c++ git glib2-devel gperf harfbuzz-devel libXcursor-devel libXi-devel libXmu-devel libXrandr-devel libopenssl-devel rpm-build ccache llvm-clang libclang autoconf213 gstreamer-devel gstreamer-plugins-base-devel gstreamer-plugins-good gstreamer-plugins-bad-devel gstreamer-plugins-ugly vulkan-loader libvulkan1`

### Void Linux { #dependencies-for-void-linux }

<!-- https://voidlinux.org/packages/ -->
<!-- Update python/servo/platform/linux_packages/xbps in servo as changes will sync to the book -->
* `sudo xbps-install {{#include linux_packages/xbps_base.txt}}`
## Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia eikä ongelmaasi ole listattu alla.

<pre><blockquote><samp>build: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.39' not found</samp></blockquote></pre>

Tämä kiertotie soveltuu, kun käännät Servoa `nix`:llä muilla Linux-jakelulla kuin NixOS:lla.
Virhe tarkoittaa, että jakelun glibc-versio on vanhempi kuin nixpkgs-paketissa.

Muuta `shell.nix`-tiedoston loppuun rivi `if ! [ -e /etc/NIXOS ]; then` muotoon `if false; then` poistaaksesi shell.nix-tuen binääritulosteille, jotka eivät riipu nix-kaupasta.
