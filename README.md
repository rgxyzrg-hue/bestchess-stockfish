# Stockfish 15.1 — corresponding source for BEST CHESS (Android)

This repository contains the complete corresponding source code of the
**Stockfish 15.1** chess engine program included in the BEST CHESS Android
app, version **1.0.0**.

- **Program:** Stockfish 15.1 (https://stockfishchess.org), by the
  Stockfish developers (see `native/stockfish-15.1/AUTHORS`).
- **Licence:** GNU General Public License version 3 (`COPYING`).
- **Upstream source:** https://github.com/official-stockfish/Stockfish,
  tag `sf_15.1`, commit `758f9c9350abee36a5865ec701560db8ea62004d`.
  `native/stockfish-15.1/` is the official source archive for that tag
  (SHA-256 of the downloaded `sf_15.1.tar.gz`:
  `d4272657905319328294355973faee40a8c28e3eecb0e7b266ed34ff33383b76`),
  extracted **without changes**.
- **Neural network:** `native/nets/nn-ad9b42354671.nnue` (CC0 1.0), the
  default network of Stockfish 15.1, embedded in the program at build time.

BEST CHESS and Stockfish are distributed as separate programs communicating
through the UCI protocol. In the app, Stockfish runs as its own process and
receives UCI text commands on standard input and answers on standard output.
This repository contains only Stockfish and what is needed to build it as
shipped. It does not contain BEST CHESS's own source code, which is not
covered by this licence.

The Stockfish developers do not endorse BEST CHESS.

## Contents

```
COPYING                       GNU GPL version 3
README.md                     this file
BUILD.md                      exact build instructions and settings
CHANGES.md                    differences from an official Stockfish build
SHA256SUMS                    checksums of every file in this repository
native/stockfish-15.1/        official Stockfish 15.1 source, unmodified
native/nets/                  the network compiled into the program (CC0)
src/CMakeLists.txt            the build file used for the app
android/build.gradle          how the app's Android build runs it
```

## Shipped program

| ABI | File in the app |
|---|---|
| arm64-v8a | `lib/arm64-v8a/libstockfish.so` |
| armeabi-v7a | `lib/armeabi-v7a/libstockfish.so` |
| x86_64 | `lib/x86_64/libstockfish.so` |

`libstockfish.so` is an ordinary executable; see `BUILD.md` for why it has
this name.
