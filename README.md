# RogueRust 4.4.0

**Oxide-first / Carbon-compatible extension framework for Rust server plugins**

RogueRust is the shared runtime and SDK used by the RogueRust plugin family. Version 4.4.0 promotes the validated 4.3.x global asset, Workshop and skin-catalogue development line while retaining the RRAM administration/UI, GridPower and performance foundations of earlier 4.x releases.

> **Compatibility rule for v4:** existing source files and established plugin-facing functionality are retained. Optimisation work remains additive or compatibility-preserving unless an API is explicitly versioned and documented.

## What RogueRust provides

RogueRust centralises work that otherwise gets duplicated across plugins: UI composition, secure callbacks, permissions/helpers, image/FileStorage handling, data and database access, HTTP, scheduling/workloads, pooling, world queries, administration helpers, profiling, resilience, integrations, update/compatibility diagnostics and shared GridPower/power-pole infrastructure.

The extension remains **Oxide-first** and supports **Carbon through its Oxide compatibility surface** without introducing a hard Carbon dependency.

## What changed in 4.4.0

RogueRust 4.4.0 consolidates the shared asset and skin infrastructure developed and live-tested through the 4.3.x line.

Key improvements include:

- Added the global RogueRust asset store at `RogueRust/Assets/RogueAssets.db` with versioned SQLite migrations and reusable Workshop/item/collection/pack metadata.
- Added `IRogueWorkshopService` for keyless known-ID Workshop metadata resolution, collection expansion, bounded batching, request deduplication and persistent cache reuse.
- Workshop metadata can now publish resolved skins into the shared skin catalogue and provide preview URLs to RogueRust ImageLibrary without each plugin maintaining its own Steam request layer.
- Added shared Steam skin-pack indexing from Rust `UnlockedViaSteamItem` relationships while keeping player entitlement and execution policy in consuming plugins.
- Expanded `IRogueSkinCatalogService` with bounded catalogue search by name, Workshop ID and content ID plus shared building-skin metadata by building grade.
- Improved skin application so held entities receive the applied skin and an immediate network refresh alongside the underlying item state.
- Retained the separation between reusable DLL infrastructure and plugin-owned player commands, permissions, blacklists, targeting, UI and skin-application policy.
- Removed an unnecessary compile-time Rust localization dependency from Workshop item-tag mapping so the protected release builds against the established Rust/Oxide reference surface.
- Aligned project, assembly, runtime, CI and public-release identity to 4.4.0.

## Module history

| Module | Available from | Purpose |
|---|---:|---|
| Core / SDK | v1.x | Shared extension lifecycle, service access and plugin contracts |
| Database & persistence | v1.x | SQLite/MySQL-compatible access, migrations and data helpers |
| RogueUI | v1.7.0 | Shared UI documents, callbacks, layouts and reusable components |
| Runtime / resilience | v2.x | Scheduling, workloads, backpressure, circuits and shutdown handling |
| Profiling & diagnostics | v2.x | Runtime metrics, health, pressure and soak diagnostics |
| World & administration | v3.x | World helpers, teleport/back, vanish and vehicle catalogues |
| F1 administration UI | v3.3.x | Shared F1 workspace, catalogues and administration primitives |
| Skin service | v3.3.11 | Shared skin catalogue, validation and effective-skin cache |
| v4 performance baseline | v4.0.0 | Compatibility-first optimisation, tooling and release overhaul |
| GridPower / power-pole backend | v4.3.1 | Shared safe public-grid power-pole discovery/state backend and diagnostics |
| Global asset store / Workshop service | v4.4.0 | Shared persistent asset, Workshop collection, Steam pack and preview metadata infrastructure |
| Building skin catalogue | v4.4.0 | Shared building-grade skin metadata for execution plugins |

## Administrator console commands

RogueRust exposes diagnostics through the server console. The current command families include:

`roguerust.version`, `roguerust.status`, `roguerust.health`, `roguerust.resilience`, `roguerust.compatibility`, `roguerust.capabilities`, `roguerust.performance`, `roguerust.recent`, `roguerust.soak`, `roguerust.pressure`, `roguerust.world`, `roguerust.jobs`, `roguerust.network`, `roguerust.database`, `roguerust.services`, `roguerust.sdk`, `roguerust.security`, `roguerust.update` and `roguerust.commands`.

Use `roguerust.commands` on a running server as the authoritative command overview for the installed build.

## Windows development

The repository has one Windows entry point:

```bat
RogueRust-Windows.cmd
```

It opens a menu for validation/build, pre-stable checks, template tests, protected builds, release packaging, reference updates and SDK-reference generation. It launches child PowerShell scripts with **process-scoped** `ExecutionPolicy Bypass`; it does not permanently change the user's or machine's PowerShell policy.

Non-interactive examples:

```bat
RogueRust-Windows.cmd -Action Validate
RogueRust-Windows.cmd -Action PreStable
RogueRust-Windows.cmd -Action Protected -Version 4.4.0
RogueRust-Windows.cmd -Action Release -Version 4.4.0
```

The existing individual scripts remain available under `scripts/` and `build/`.

## Building

RogueRust targets .NET Framework 4.8 and requires current Rust/Oxide reference assemblies in `References/`. Use the Windows launcher to update references and validate the repository before producing a release.

The protected release pipeline remains the canonical packaging path. Do not publish a DLL that has not passed repository validation, compatibility/API checks and release verification.

## Plugin and mod development

Developer material is kept in the source repository so public releases can remain focused on server administrators.

- `docs/` — SDK and implementation guides.
- `samples/` — example consumers and integration patterns.
- `templates/` — starting points for RogueRust-aware plugins.
- `specs/` — compatibility and behavioural specifications.
- `FEATURES.md` — detailed capability inventory.
- `CHANGELOG.md` — version history and testing notes.

When extending the framework, prefer shared services over duplicating expensive timers, world scans, network requests, persistence loops or UI infrastructure in individual plugins.

## Release channels

- **Stable:** `vX.X.X` and `update-manifest.json`.
- **Testing:** `vX.X.X-testing` and `update-manifest-testing.json`.

For this release line, testing builds resolve to **4.4.0-testing** and remain isolated on `update-manifest-testing.json`. Stable **4.4.0** is published from `main` through the protected release workflow and updates only `update-manifest.json`.

## 4.4.0 release validation

Validate clean Oxide startup, Carbon compatibility through the Oxide compatibility surface, RogueRust plugin-family loading, Workshop/cache persistence, skin catalogue/search, held-item visual refresh, building-skin catalogue consumers, GridPower, shutdown/restart behaviour, updater channel isolation and the protected CI artifact.