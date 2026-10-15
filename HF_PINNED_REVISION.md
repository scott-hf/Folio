# HF pinned revision

This is the Hello Friend fork of Folio, consumed as a git submodule of the private
`HF_Plugins` repo (`Engine/Plugins/HF_Plugins/Folio`) in the AngelScript Unreal Engine fork.

- Upstream: https://github.com/Fogspire/Folio (branch `main`)
- Fork: https://github.com/scott-hf/Folio (branch `hf-5-8`, the branch HF_Plugins tracks)
- Pinned upstream SHA: `2e23916b04e80fa9f8e4af5f6af9f6a27f797fb2` (upstream commit dated 2026-10-14)
- Pinned on: 2026-10-15 (WI-1580 t100)

## Why pinned

The PrjCruise Folio screens (WI-1574 pilot, WI-1580 batch 1) were built and validated against this
exact revision (`test Folio` 16/16, uipreview strict + a11y). Folio is young and moves fast; an
unreviewed upstream bump can change RML/RCSS behaviour or the UFolioWidget API under every
consuming project. The fork exists so the plugin lives in the shared engine location
(`HF_Plugins`) instead of per-project copies.

## Carried patches (on top of the pinned upstream SHA)

- WI-1580 t112: `Source/Folio/Folio.Build.cs` adds `RuntimeDependencies` for `$(PluginDir)/rcss/...` and
  `$(PluginDir)/fonts/...` (UFS), and `FolioDocumentPath.cpp` `IsStaged()` treats a file under the plugin
  root's `rcss`/`fonts` as staged. Reason: as an engine-fork plugin Folio sits outside the project, so the
  Content-relative `DirectoriesToAlwaysStageAsUFS` cannot name its `/folio/` root, and `Folio.Document.Validation`
  failed ("a staged document is not warned about"). Upstream (project-local plugin) works via
  `../Plugins/Folio/*` staging lines. Candidate to offer upstream (WI-1579).

## Rules

- Never push to Fogspire. Push only to `scott-hf/Folio`.
- `hf-5-8` carries upstream plus HF-only additions (this file, `HF_BUILD.md`). Keep HF changes
  additive and small so rebasing stays trivial.
- Native libs (`Source/ThirdParty/*/lib`, `include`) and `bin/` are gitignored upstream and
  are built per machine: see `HF_BUILD.md`.

## How to rebase onto a newer upstream

1. `git remote add upstream https://github.com/Fogspire/Folio.git` (once), `git fetch upstream`.
2. Review `git log 2e23916b..upstream/main` for API, RML/RCSS and CMake changes.
3. `git checkout hf-5-8 && git rebase upstream/main` (only the HF-only commits replay).
4. Update the pinned SHA and date above.
5. Rebuild the libs on one machine (`HF_BUILD.md`), run `test Folio` and the PrjCruise Folio
   screens, then `git push --force-with-lease origin hf-5-8`.
6. In `HF_Plugins`: `git -C Folio fetch && git -C Folio checkout <new hf-5-8 SHA>`, commit and push
   HF_Plugins first, then bump the engine repo's HF_Plugins pointer.
