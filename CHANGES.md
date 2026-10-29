# Differences from an official Stockfish 15.1 build

## Source changes (from BEST CHESS 1.1.0, 2026-10-29)

Two lines of Stockfish's source are changed, each marked with a
`BEST CHESS change` comment. Nothing else in the source is added,
removed or edited. Version 1.0.0 used the unmodified source (git tag
`bestchess-1.0.0`).

1. `native/stockfish-15.1/src/search.cpp`, `UCI::pv()`: the elapsed
   time used to print nodes per second is at least 1 ms
   (`std::max(Time.elapsed() + 1, TimePoint(1))`). When the system
   clock stepped back by a moment (seen on x86 Android emulators), the
   elapsed time could be 0 and the division stopped the program with an
   arithmetic exception. Searches and moves are unchanged.
2. `native/stockfish-15.1/src/uci.cpp`, `bench()`: the same guard for
   the benchmark's nodes-per-second line.

## Build differences


The differences are only in how the program is built:

1. **Build system:** CMake with the Android NDK (`src/CMakeLists.txt`)
   instead of Stockfish's Makefile. It uses the same compiler definitions
   and flags as the Makefile's `COMP=ndk` builds for each architecture (see
   `BUILD.md`).
2. **File name:** the program is named `libstockfish.so` so that Android
   installs it into the app's native library folder. It is still an
   ordinary executable with Stockfish's own `main()`.
3. **Linking:** static C++ runtime and 16 KB page alignment, as Android and
   Google Play require for current devices.

The network is embedded exactly as in official builds (Stockfish's
`incbin`).

Date of this build configuration: 2026-10-07.
