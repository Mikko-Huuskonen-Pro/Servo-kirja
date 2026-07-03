# Kääntäminen macOS:lle

- Lataa ja asenna [Xcode](https://developer.apple.com/xcode/) ja [`brew`](https://brew.sh/).
- Asenna `uv`: `curl -LsSf https://astral.sh/uv/install.sh | sh` 
- Asenna `rustup`: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- Käynnistä shell uudelleen varmistaaksesi, että `cargo` on käytettävissä
- Asenna muut riippuvuudet: `./mach bootstrap`
- Käännä servoshell: `./mach build`

## Vianmääritys

Katso [Yleinen vianmääritys](general-troubleshooting.md) -osio, jos käännöksessä on ongelmia.
