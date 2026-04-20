# PAC-MAN CE DX — PS Vita Port

A homebrew `.so`-loader port of **PAC-MAN Championship Edition DX** (Android) to the PlayStation Vita, plus a browser-based installer that turns your user-supplied APK + OBB into a Vita-ready VPK.

> **This repository contains only the loader and the website. It does not contain, distribute, or link to PAC-MAN CE DX itself.** The game, its assets (`assets/…`), and the binaries `libPacmanCE.so` / `libfmod.so` are © Bandai Namco Entertainment. To play, you must legally own the Android version and supply the `.apk` and `.obb` from your own copy.

## What's in here

| Path | What it is |
| --- | --- |
| `source/` | Vita-side loader: relocates the Android `.so`, stubs Android/Java APIs, bridges OpenGL ES, etc. |
| `lib/vitaGL/` | Custom fork of [vitaGL](https://github.com/Rinnegatamante/vitaGL) with patches needed by this game (softfp ABI for `.so` interop, plus targeted shader/draw fixes). |
| `lib/vitaGL_clean/` | Pristine vitaGL build (hardfp) used by the standalone settings configurator. |
| `lib/{falso_jni,fios,kubridge,libc_bridge,sha1,so_util}/` | Standard `.so`-loader support libraries. |
| `extras/livearea/` | Vita LiveArea theme (icon, background, template). |
| `docs/` | The website. Drop in your APK + OBB → get a VPK + data zip in your browser. Nothing leaves your machine. |
| `.github/workflows/` | CI that builds the loader VPK and publishes the site to GitHub Pages on every push to `main`. |
| `build.sh` | Convenience wrapper: `./build.sh [extra cmake -D flags…]` |

## For end users

Don't build from source — just visit the [project website](https://mcallbosco.github.io/pacman-ce-dx-vita/) and drop in your APK + OBB. You'll get back two files:
- `pacmancedx.vpk` — install via VitaShell
- `pacmancedx-data.zip` — unzip and FTP to `ux0:data/pacmancedx/`

## For developers — building the loader

Requirements: [VitaSDK](https://vitasdk.org/) with the `VITASDK` env var pointing at the install root.

```sh
./build.sh
# → build/pacmancedx.vpk  (loader-only, ~2 MB; no game data)
```

`build.sh` is a thin wrapper around the standard CMake invocation; any `-D` flags you pass get forwarded:

```sh
./build.sh -DENABLE_RUNTIME_LOGS=ON -DVITA_MSAA_MODE=4X
# or, equivalently:
cmake -B build -S . -DENABLE_RUNTIME_LOGS=ON -DVITA_MSAA_MODE=4X
cmake --build build -j$(nproc)
```

### Build flags

#### Rendering

| Flag | Values | Default | Effect |
| --- | --- | --- | --- |
| `VITA_MSAA_MODE` | `OFF`, `2X`, `4X` | `OFF` | vitaGL multisample antialiasing. `OFF` is the default for the released VPK to maximize GPU headroom; users can opt into `2X` or `4X` from the in-game configurator. |
| `SHADER_FORMAT` | `GLSL`, `CG`, `GXP` | `GLSL` | Source format for the game's shaders. `GXP` skips runtime compilation (fastest cold start) but requires pre-compiled `.gxp` artifacts in `data/`. |
| `DUMP_COMPILED_SHADERS` | `ON`, `OFF` | `OFF` | When using `GLSL` or `CG`, persist compiled shaders to disk so subsequent launches skip compilation. Ignored for `GXP`. |

#### Runtime / IO

| Flag | Values | Default | Effect |
| --- | --- | --- | --- |
| `USE_SCELIBC_IO` | `ON`, `OFF` | `ON` | Route file/string IO through `SceLibcBridge`. |

#### Diagnostics (all default `OFF`)

| Flag | Effect |
| --- | --- |
| `ENABLE_RUNTIME_LOGS` | Verbose `.so`-loader log to `ux0:data/pacmancedx/debug.log`. |
| `ENABLE_GL_DEBUG_HOOKS` | Heavy OpenGL call instrumentation. |
| `ENABLE_FALSOJNI_VERBOSE` | Sets `FALSOJNI_DEBUGLEVEL=0` so every JNI call and lookup is logged. |
| `ENABLE_AUDIO_LOGS` | FMOD / BGM / audio diagnostics. |
| `ENABLE_IO_PROFILING` | Per-call IO profiling. |

#### Packaging

| Flag | Values | Default | Effect |
| --- | --- | --- | --- |
| `BUNDLE_GAME_DATA_DIR` | path | empty | If set, recursively bundles the directory into the VPK at `data/`.  |
| `DATA_PATH` | path | `ux0:data/pacmancedx/` | Where the loader looks for game data on the Vita. |
| `WRITABLE_PATH` | path | `ux0:data/pacmancedx/` | Where saves, config, log, and shader cache go. Must be on `ux0:`. |

## Licensing

| Component | License | Holder |
| --- | --- | --- |
| Loader (`source/`, `extras/`, `docs/`, build glue) | **GPL-3.0** | This project |
| `lib/vitaGL/` (modifications) | **LGPL-2.1** (inherited) | Rinnegatamante + contributors + this project |
| `lib/vitaGL_clean/` | **LGPL-2.1** (upstream) | Rinnegatamante + contributors |
| `lib/{falso_jni,kubridge,so_util,…}/` | Various — see each subdir's headers | Original authors |
| **PAC-MAN CE DX game + assets** | **All rights reserved** | © Bandai Namco Entertainment — **not in this repo** |

Full GPL-3.0 text in `LICENSE`. The loader makes no attempt to bypass DRM and is useless without a legally obtained copy of the Android game.

## Credits

- **Rinnegatamante** — vitaGL, the `.so`-loader pattern this port follows, countless reference ports
- **TheFloW** — original `.so`-loader research on Vita
- **VitaSDK contributors** — toolchain + system bindings
- **Mcall** — this port
