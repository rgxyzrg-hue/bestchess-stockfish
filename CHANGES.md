# Differences from an official Stockfish 15.1 build

Stockfish's source code is **not modified**. No file is added to it,
removed from it or edited.

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
