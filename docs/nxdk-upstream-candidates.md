# nxdk upstream candidates

Assessment: 2026-11-18. Priority means suggested investigation/contribution order,
not readiness to merge. These are technical fit judgments, not maintainer approval.
Game changes are covered separately in [fork differences](fork-differences.md).

[nxdk’s scope](https://github.com/XboxDev/nxdk/blob/master/README.md) is reusable
original Xbox tooling, runtime, drivers, and graphics support.

## Prioritized candidate table

| Priority | Category | Candidate | Why it fits upstream | Evidence / remaining work |
| --- | --- | --- | --- | --- |
| **P1 · first** | Graphics correctness | **VBlank queue retirement:** retain a ready frame until scanout actually selects it. | Repairs a general pbkit presentation invariant. | Local patch and queue/scanout observations exist. Extract a minimal reproducer and rebase the narrow fix onto current nxdk. |
| **P1 · paired** | Graphics API / ownership | **Safe back-buffer acquisition:** prevent drawing into the displayed surface. | Applications should have a clear surface-ownership contract. | Game-side guard exists; it exposed stalls with old queue retirement. Review both together, including interrupt synchronization and progress guarantees. Decide whether the result is an API, implementation fix, or documented contract. |
| **P2 · investigate** | Graphics robustness | **Push-buffer capacity and wrap/reset handling.** | A reproducible capacity-handling defect could affect other titles. | A 1 MiB ring avoids reported game startup hangs. That workaround does not prove the precise SDK defect; reproduce bounds/wrap behavior independently. |
| **P3 · enhancement** | Build integration | **Minimal CMake toolchain/sample**, including Windows C++ header isolation. | Makes the SDK easier to consume across host platforms. | Extract from the game toolchain, remove game dependencies, and validate supported hosts. Our incorrect compiler/linker invocation alone is not an SDK bug. |
| **P3 · enhancement** | Executable tooling | **`cxbe` title-ID configuration.** | Removes post-conversion certificate patching for homebrew save identities. | Local patching script exists. Verify current converter options before proposing a new one; this is a feature, not evidence of corrupt output. |
| **P4 · verify need** | Library compatibility | **`strcasecmp` / `strncasecmp` support.** | Potentially useful compatibility APIs for other ports. | Check current nxdk/PDCLib support and intended POSIX scope. Use proper declarations/types and library tests rather than copying game shims unchanged. |

## Evidence and submission gates

| Area | What is established | What must not be inferred / next step |
| --- | --- | --- |
| Queue implementation | The [upstream source](https://github.com/XboxDev/nxdk/blob/master/lib/pbkit/pbkit.c) checked for this assessment retires ready entries unconditionally. [Local patch notes](../port/src/xbox/PBKIT_PATCH.md) explain the invariant. | A candidate implementation still needs isolated reproduction and review against current nxdk. |
| Presentation investigation | Recorded overlapping render/scanout surfaces and queue-loss states. | The [validation record](480i-presentation-validation.md) includes an incomplete final-candidate run and no reproduction of the exact physical moving band. Do not claim a universally qualified 480i cure. |
| Version compatibility | The local core derives from `fb5a9a7`; the installed SDK has later PTIMER changes. | Submit the small fix on current nxdk. Replacing its core with our old copy would undo unrelated improvements. |
| Reproducer | Game diagnostics provide starting evidence. | Use a bounded SDK-only trigger, expected contract, and before/after observations. Remove game diagnostic switches and `g_SPXB` telemetry names. |
| Hardware | Existing notes describe bounded prior observations. | This document adds no runtime tests. Follow shared Xbox hardware-testing instructions for new console validation. |

## Application decisions to keep in the game

| Category | Decision | Boundary with nxdk |
| --- | --- | --- |
| Resource policy | 1 MiB ring, texture budgets, effect reductions, frame cap | Keep chosen values per game. Extract an independently demonstrated SDK defect separately. |
| Display policy | Dashboard mode selection, 720p fallback | Game behavior unless an SDK mode-handling defect is reproduced. |
| Dependency integration | Allocators required by freestanding `Z_SOLO` zlib | Fulfill the dependency contract; a separate implementation/documentation defect needs its own evidence. |
| Startup integration | Correct CRT entry and linker flags | Correct application setup; reusable documentation may still be valuable. |
