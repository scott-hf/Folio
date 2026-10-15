# HF per-machine build of Folio's native pieces

Folio's static libs (RmlUi, msdfgen) and tools (`uipreview.exe`, `msdfbake.exe`) are gitignored
upstream, so a fresh checkout of this submodule has no `Source/ThirdParty/*/lib` and the Folio
module will not link. Every machine that builds the engine must run the steps below once
(and again after changing the pinned revision, a patch under `patches/`, or the engine FreeType).

Pinned upstream revision and rebase notes: `HF_PINNED_REVISION.md`.

## Prerequisites

- Windows, Visual Studio 2022 with the C++ workload. CMake and Ninja are taken from inside VS
  by `scripts/vsenv.sh`; nothing needs to be on PATH.
- Git Bash (`bash`) to run the `.sh` scripts.
- Network access on the first run: RmlUi/msdfgen are built from CMake FetchContent dependencies (`_deps` under `build/`).
- The engine's bundled FreeType 2.14.1 (`Engine/Source/ThirdParty/FreeType2/FreeType2-2.14.1`)
  must exist; override with `-DFOLIO_FREETYPE_VERSION=...` if the engine ships another.

## Commands

Run from the Folio checkout (`<engine>/Engine/Plugins/HF_Plugins/Folio`):

```
bash scripts/build_ue_libs.sh --engine-dir <engine>/Engine
```

- `--engine-dir` is the `Engine` directory itself (`E:/UE/UnrealEngine-Angelscript-Scott/Engine`),
  not the repo root: the script appends `Source/ThirdParty/FreeType2/...` to it.
- Takes several minutes (the 2026-10-15 run finished inside 9 minutes). After a failed or wrong-`--engine-dir` run add `--clean`: the CMake cache
  keeps the first `FOLIO_ENGINE_DIR` and the same error repeats otherwise.

Tools for the preview/bake pipeline (needed for authoring and validation, not for a game build):

```
bash scripts/build.sh --target uipreview
bash scripts/build.sh --target msdfbake
```

or, on a machine that already has them (e.g. a PrjCruise checkout with `Plugins/Folio/bin/win64`),
copy the two `.exe` files into `bin/win64/`. Do not copy the `.ilk`/`.pdb`.

## Expected outputs (measured 2026-10-15, VS 2022 toolset 14.44)

| File | Size |
|---|---|
| `Source/ThirdParty/RmlUiLibrary/lib/Win64/rmlui.lib` | ~84 MB |
| `Source/ThirdParty/RmlUiLibrary/lib/Win64/rmlui_debugger.lib` | ~4.4 MB |
| `Source/ThirdParty/MsdfgenLibrary/lib/Win64/msdfgen-core.lib` | ~4.4 MB |
| `Source/ThirdParty/MsdfgenLibrary/lib/Win64/msdfgen-ext.lib` | ~0.4 MB |
| `Source/ThirdParty/MsdfgenLibrary/lib/Win64/skia.lib` | ~5.5 MB |
| `bin/win64/uipreview.exe` | ~9.3 MB |
| `bin/win64/msdfbake.exe` | ~7.0 MB |

`Source/ThirdParty/*/include` is staged too. The scratch tree `build/` (~350 MB) can be deleted.

All of the above are gitignored by Folio's own `.gitignore`: `git status --short` inside
`Folio/` must stay empty after the build. Never `git add -f` them.

## hf-onboard

This is a manual step today. `hf-onboard` needs a phase that, per machine: checks that
`Engine/Plugins/HF_Plugins/Folio/Source/ThirdParty/RmlUiLibrary/lib/Win64/rmlui.lib` exists, and if not
runs `scripts/build_ue_libs.sh` (and copies or builds the two tools). Until that phase exists, a
machine that pulls the engine without it fails to link the Folio module.
