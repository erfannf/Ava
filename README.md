<p align="center">
  <img
    src="docs/ava-hero.png"
    alt="Ava media controls above a monochrome abstract sculpture"
    width="100%">
</p>

# Ava

Ava is a native Windows 11 productivity app built with Qt 6, QML, C++20, and
Win32. It pairs a compact live-activity island with a separate native Codex
workspace.

| Application | Purpose |
| --- | --- |
| **Ava** | Island, launcher, media controls, system monitor, timer, wallpaper tools, Liquid Glass, and optional window tiling. |
| **AvaChat** | Native Codex conversations, approvals, attachments, diffs, and Git workflows. |

## Highlights

- Live Windows media, audio, battery, network, clock, calendar, and system state.
- Fast app, URL, file, and folder launching with `Ctrl+K`.
- Native optional Liquid Glass rendered through Windows Graphics Capture and
  Direct3D 11, with no embedded browser runtime.
- Cider enrichment for queue, playlists, search, history, lyrics, and audio
  pulse, with Windows media controls as the fallback.
- Reduced-motion support, native tray controls, persistent settings, and
  optional Dwindle window tiling.
- A separate native AvaChat client backed by the installed Codex app-server.

## Build

### Requirements

- Windows 11
- Visual Studio 2022 with the C++20 MSVC toolchain
- A Windows SDK that provides C++/WinRT headers (10.0.22000 or later)
- CMake 3.21+
- Qt 6.5+ with Core, Concurrent, Gui, Network, Qml, Quick, Quick Controls,
  Quick Dialogs, Quick Layouts, Shader Tools, Test, and Widgets

Only the MSVC toolchain is supported. Ava links directly against Direct3D 11,
DWM, PDH, Windows Core Audio, and WinRT capture APIs.

### Install the toolchain

If you already have a Qt-enabled developer shell, skip to *Configure and build*.

Visual Studio 2022 Build Tools are enough; the full IDE is not required. Install
the *Desktop development with C++* workload, then CMake:

```powershell
winget install --id Kitware.CMake --exact
```

Qt can be installed without a Qt account using `aqtinstall`. `qtshadertools` is
an add-on module and must be requested explicitly; the remaining modules ship
with the base package:

```powershell
python -m pip install aqtinstall
python -m aqt install-qt windows desktop 6.8.3 win64_msvc2022_64 -m qtshadertools -O C:\Qt
```

### Configure and build

Point CMake at the Qt kit unless `qmake` is already on `PATH`:

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DCMAKE_PREFIX_PATH="C:/Qt/6.8.3/msvc2022_64"
cmake --build build --config Release --parallel
.\build\Release\Ava.exe
.\build\Release\AvaChat.exe
```

Running from `build\Release` requires the Qt `bin` directory on `PATH`.

### Deploy a standalone build

The install target copies both executables and runs `windeployqt`, producing a
self-contained tree that runs without Qt on `PATH`:

```powershell
cmake --install build --config Release --prefix .\dist
.\dist\bin\Ava.exe
```

The result is `dist\bin` (executables and Qt DLLs), `dist\plugins`, and
`dist\qml`, wired together by a generated `dist\bin\qt.conf`. `dist/` is
ignored by Git.

### Tests

Ensure the active Qt `bin` directory and `build/Release` are on `PATH`, then:

```powershell
ctest --test-dir build -C Release --output-on-failure
```

The authenticated Codex end-to-end test is opt-in through
`AVA_RUN_LIVE_CODEX_TEST=1` because it uses the current account and workspace.

### Runtime dependencies

AvaChat and the island's Codex panel require the Codex CLI, installed and
signed in, at a version that provides `codex app-server`:

```powershell
npm install -g @openai/codex
codex login
```

Ava discovers the CLI through `AVA_CODEX_EXECUTABLE` first, then the global npm
package, common shim locations, and finally `PATH`. Git must be on `PATH` for
worktrees and the Git change center. An authenticated GitHub CLI (`gh`) is
needed only to create pull requests from AvaChat.

### Notes and known issues

- **The Liquid Glass shader is split across two adjacent string literals.**
  MSVC truncates any single string literal longer than roughly 16,380
  characters (error C2026). The HLSL pixel shader in
  `src/liquidglasscaptureworker.cpp` exceeds that, so it is written as two
  adjacent literals, which the compiler concatenates after the per-literal
  check. Keep the split when editing the shader; merging the halves back into
  one literal breaks the build.
- **Smart App Control blocks locally built binaries.** Ava is unsigned, so a
  machine with Smart App Control enabled refuses to start it with *"An
  Application Control policy has blocked this file"*. Verify with
  `Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\CI\Policy"`, where
  `VerifiedAndReputablePolicyState` is `0` for off, `1` for enforcing, and `2`
  for evaluation. Self-signing does not help, because Smart App Control
  requires a reputable publisher signature. Turning it off is permanent until
  Windows is reinstalled.

## License

Ava's original source code is available under the
[PolyForm Noncommercial License 1.0.0](LICENSE). Commercial use requires
separate written permission from the project owner.

Bundled third-party fonts and icons retain their own licenses:

- Inter: `assets/fonts/OFL-Inter.txt`
- Geist Mono: `assets/fonts/OFL-Geist.txt`
- Microsoft Fluent UI System Icons: `assets/icons/LICENSE`
