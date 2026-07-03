# Kääntäminen NixOS:lle

- [Asenna Nix](https://nixos.org/download), paketinhallinta.
  Helpoin tapa on käyttää asennusohjelmaa monikäyttäjä- tai yksikäyttäjäasennuksella (valintasi mukaan).
- Kerro `mach`:lle Nixin käytöstä: `export MACH_USE_NIX=`
- Kirjoita `nix-shell` päästäksesi shelliin, jossa on kaikki tarvittavat työkalut ja riippuvuudet.
- Asenna muut riippuvuudet: `./mach bootstrap`
- Käännä servoshell: `./mach build`

## Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia eikä ongelmaasi ole listattu alla.


<pre><blockquote><samp>error: <a href="https://github.com/NixOS/nix/blob/e3fa7c38d7af8f34de0c24766b2e8cf1cd1330f0/src/libutil/file-system.cc#L164-L184">getting status of</a> /nix/var/nix/daemon-socket/socket: Permission denied</samp></blockquote></pre>

Jos saat tämän virheen ja olet asentanut Nixin järjestelmän paketinhallinnalla:

- Lisää itsesi `nix-users`-ryhmään
- Kirjaudu ulos ja takaisin sisään

<pre><blockquote><samp>error: <a href="https://github.com/NixOS/nix/blob/e3fa7c38d7af8f34de0c24766b2e8cf1cd1330f0/src/libexpr/eval.cc#L2849">file 'nixpkgs' was not found in the Nix search path (add it using $NIX_PATH or -I)</a></samp></blockquote></pre>

Tämä virhe on vaaraton, mutta voit korjata sen näin:

- Aja `sudo nix-channel --add https://nixos.org/channels/nixpkgs-unstable nixpkgs`
- Aja `sudo nix-channel --update`
