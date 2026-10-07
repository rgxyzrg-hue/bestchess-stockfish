# Building the Stockfish program shipped in BEST CHESS

This rebuilds `libstockfish.so`, the Stockfish 15.1 program included in the
BEST CHESS Android app, from the files in this repository.

## What you need

- Android NDK **28.2.13676358**
- CMake **3.22.1** or newer, and Ninja (both come with the Android SDK's
  CMake package)

## Build (one command per Android ABI)

Replace `<NDK>` with the NDK folder. Run these from the repository root.

```
cmake -S src -B out/arm64-v8a -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=<NDK>/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-24 -DSTOCKFISH_VERSION=15.1
cmake --build out/arm64-v8a
```

For the other ABIs, use `-DANDROID_ABI=armeabi-v7a` or
`-DANDROID_ABI=x86_64` (emulator only), each with its own `-B out/<abi>`
folder.

The result is `out/<abi>/libstockfish.so`. It is an ordinary Android
executable (position-independent), despite the `.so` name. Android only
installs files named `lib*.so` into an app's native library folder, the one
place an app may run its own program from.

The app's build uses the same CMake file through the Android Gradle plugin
(`android/build.gradle`). That build compiles with debug information and
removes it when packaging; the program's behaviour is the same.

## Settings used (same as Stockfish's Makefile, `COMP=ndk`)

| ABI | Defines and flags | Makefile equivalent |
|---|---|---|
| arm64-v8a | `IS_64BIT USE_POPCNT USE_NEON=8` | `ARCH=armv8` |
| armeabi-v7a | `USE_POPCNT USE_NEON=7`, `-mfpu=neon` | `ARCH=armv7-neon` |
| x86_64 | `IS_64BIT USE_POPCNT USE_SSE41 USE_SSSE3 USE_SSE2`, `-msse4.1 -mpopcnt` | `ARCH=x86-64-sse41-popcnt` |
| all | `NDEBUG USE_PTHREADS`, `-O3 -fno-exceptions`, C++17, static C++ runtime, `-Wl,-z,max-page-size=16384` | |

## The network

`native/nets/nn-ad9b42354671.nnue` (CC0) is Stockfish 15.1's default
network. The build copies it next to the compiler's working folder, and
Stockfish's own `incbin` embeds it in the program, as in official builds.

## Quick check on a device or emulator (optional)

```
adb push out/arm64-v8a/libstockfish.so /data/local/tmp/stockfish
adb shell chmod 755 /data/local/tmp/stockfish
adb shell "printf 'uci\nisready\ngo depth 10\n' | /data/local/tmp/stockfish"
```

The program prints `id name Stockfish 15.1`, `uciok`, `readyok`, search
information and a `bestmove` line.
