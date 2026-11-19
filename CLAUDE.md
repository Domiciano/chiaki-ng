# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

chiaki-ng is an open source PlayStation 4/5 Remote Play client (successor to Chiaki) for Linux, Windows, macOS, Android, Nintendo Switch and Steam Deck. See also `CONTRIBUTOR_GUIDE.md` (orientation and "where to start" per bug type) and `.github/copilot-instructions.md`.

## Build and test

Submodules are required (munit, nanopb, jerasure, gf-complete, curl, cpp-steam-tools, borealis, oboe):

```bash
git submodule update --init --recursive
```

Desktop build (CMake + Ninja):

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --target chiaki        # Qt GUI
cmake --build build --target chiaki-unit   # unit tests (CHIAKI_ENABLE_TESTS is ON by default)
```

On macOS, mirror the flags used in `.github/workflows/build-pr.yaml` (Homebrew `qt@6`, `openssl@3`, `protobuf@29`; `-DCHIAKI_ENABLE_CLI=OFF -DCHIAKI_ENABLE_STEAMDECK_NATIVE=OFF`; explicit `OPENSSL_*` paths and `CMAKE_PREFIX_PATH`; `CPATH=$(brew --prefix)/opt/ffmpeg/include`). `openssl@4` headers in the Homebrew prefix shadow `openssl@3` and break the bundled curl, so unlink it.

Tests use munit (`test/`), all built into one `chiaki-unit` binary:

```bash
ctest --test-dir build --output-on-failure   # registered as a single ctest "unit"
build/test/chiaki-unit                       # all suites
build/test/chiaki-unit /http                 # one suite (prefix match)
build/test/chiaki-unit /http/response_parse  # one test
build/test/chiaki-unit --list                # list test names
```

New test files must be added to `CHIAKI_UNIT_SOURCES` in `test/CMakeLists.txt` and their `MunitTest[]` array registered as a suite in `test/main.c`. `aia.c` is excluded under mbedTLS and `ffmpegdecoder.c` requires the FFmpeg decoder. Unit coverage is small; changes to streaming, rendering, audio devices or session setup still need manual testing against a console.

## Architecture

- **`lib/` (`chiaki-lib`, C)**: the Remote Play protocol core, shared by every frontend. Public headers in `lib/include/chiaki/`. Session negotiation in `ctrl.c`/`session.c`, transport in `takion.c`, A/V receive/reassembly in `videoreceiver.c`/`audioreceiver.c` (with `fec.c`, `reorderqueue.c`), codecs in `ffmpegdecoder.c`/`opusdecoder.c`/`opusencoder.c`, input back to the console in `controller*.c`/`feedbacksender.c`/`audiosender.c`. PSN/internet play (holepunch, RUDP) lives in `lib/src/remote/`. Protobuf messages are generated at build time from `lib/protobuf/takion.proto` via nanopb; never commit the generated `takion.pb.c/.h`.
- **`gui/` (Qt6 + QML, C++)**: desktop app wrapping `chiaki-lib`. `qmlbackend.cpp` orchestrates app state, discovery, PSN token refresh and session lifecycle; `streamsession.cpp` turns `Settings` + host credentials into `ChiakiConnectInfo`, sets up decoders/audio/haptics/controllers and handles stream callbacks; `qmlmainwindow.cpp` renders decoded frames through libplacebo/Vulkan with the QML overlay composited on top; `settings.cpp`/`qmlsettings.cpp` hold persistent settings (behaviour is settings-driven, so don't hardcode defaults in session code). QML views are in `gui/src/qml/`.
- **`cli/`**: small `discover`/`wakeup` wrapper over `chiaki-lib`, also reachable from the GUI binary's command mode (`gui/src/main.cpp`).
- **`setsu/`** (touchpad via udev/evdev) and **`steamdeck_native`** (gyro/haptics via HIDAPI): optional Linux input libraries.
- **`android/`** (Gradle, reuses the root `CMakeLists.txt` with GUI/CLI/FFmpeg disabled) and **`switch/`** (Borealis UI): separate frontends on the same lib.

Video path: packets in `lib` → `videoreceiver.c` → `ffmpegdecoder.c` → frame handed to GUI → libplacebo render in `qmlmainwindow.cpp`. When debugging, first decide whether a problem is protocol/session, receive/decode timing, renderer/audio device, or QML/UI.

## Conventions

- C code in `lib/` uses paired `*_init`/`*_fini` lifecycles and returns `ChiakiErrorCode`; keep that pattern.
- Respect feature gates in CMake and source (`CHIAKI_ENABLE_*`, `CHIAKI_LIB_ENABLE_*`, `CHIAKI_GUI_ENABLE_*`). Many options are tri-state `AUTO|ON|OFF` (`tri_option` in `CMakeLists.txt`), not booleans. Keep platform code behind existing guards (`Q_OS_*`, `WIN32`, `CHIAKI_GUI_ENABLE_STEAMDECK_NATIVE`, `CHIAKI_GUI_ENABLE_SETSU`, ...).
- Source files carry an `SPDX-License-Identifier: LicenseRef-AGPL-3.0-only-OpenSSL` header.
- Avoid editing vendored code (`third-party/`, `switch/borealis/`, `test/munit/`, downloaded FFmpeg/SDL trees) unless required.
- A dependency, version or build-option change must be kept consistent across `CMakeLists.txt`, `scripts/` (build-*.sh, `scripts/flatpak/` manifests) and `.github/workflows/`, in the same change. Renderer changes in particular often touch QML, C++, CMake, Flatpak and CI pins together.
- In QML, input/focus handlers usually need to be on focusable `Item`s rather than the top-level `Window`.

## Web version (`web/`)

A web version (React/TS frontend + C++ bridge over `chiaki-lib`, WebRTC to the browser, deployed with docker-compose) is being specified in `web/specs/`. It has its own semver in `web/VERSION`. Each `web/specs/vX.Y.Z/` holds `functional/` (FS-NN), `technical/` (TS-NN) and a `devprogress.md` to update as tasks progress. Conventions are in `web/specs/README.md`.

## Packaging, release and docs

- Version is set in `CMakeLists.txt` (`CHIAKI_VERSION_MAJOR/MINOR/PATCH`). Release notes go in `doc/release_notes.md`; the user-facing site is MkDocs (`mkdocs.yml`, `docs/`, with release history in `docs/updates/`).
- Packaging: AppImage (`scripts/build-appimage.sh`, which also runs `chiaki-unit`), Flatpak (`scripts/flatpak/`), macOS app/DMG (`scripts/macos-*.sh`; arm64 only, minimum macOS follows the build machine), Windows via MSYS2 (`scripts/deploy-windows-msys2.sh`, `scripts/chiaki-ng.iss`). CI for PRs is `.github/workflows/build-pr.yaml`; releases use `build-release.yaml`.
