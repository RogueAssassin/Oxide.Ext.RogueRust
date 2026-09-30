# RogueRust Changelog

## 4.4.0

### Global assets and Workshop infrastructure
- Promoted the validated 4.3.x global asset and Workshop development line to the stable 4.4.0 release.
- Added the shared `IRogueAssetStoreService` backed by `RogueRust/Assets/RogueAssets.db` with versioned SQLite migrations for reusable asset, Workshop item, collection and Steam skin-pack metadata.
- Added `IRogueWorkshopService` for keyless known-ID Workshop metadata resolution, seven-day cache freshness, bounded batches, in-flight request deduplication, collection expansion and persistent reuse.
- Workshop records can publish resolved skins into the shared skin catalogue and provide cached preview URLs to RogueRust ImageLibrary.
- Added Steam skin-pack indexing from Rust `UnlockedViaSteamItem` relationships while keeping ownership/entitlement decisions in consuming plugins.

### Skin and building platform
- Added bounded shared skin-catalogue search by name, Workshop ID and content ID so execution plugins no longer need duplicate catalogue indexes.
- Added shared building-skin metadata by grade, including the current Rust Wood, Stone, Metal and Armored building skin families used by RogueRust skin consumers.
- Improved shared skin application so held entities receive the selected skin and an immediate network update alongside the underlying item.
- Kept player-facing commands, permissions, blacklists, targeting, UI, selections and application policy in execution plugins such as RogueRustSkins.

### Release and compatibility
- Removed the Workshop item-tag lookup's compile-time dependency on Rust localization display-name types, restoring compatibility with the established protected Rust/Oxide reference set.
- Aligned project, assembly, runtime, CI and public-release identities to 4.4.0.
- Stable and testing updater channels remain isolated: `update-manifest.json` for stable and `update-manifest-testing.json` for testing.
- Protected release publication updates the public release assets and stable updater pointer used by RogueRust automatic updates.


## 4.3.3

- Added a shared skin-catalogue search API so execution plugins can query the DLL catalogue by skin name, Workshop ID or content ID without maintaining duplicate search indexes.
- Began the contextual skinning development line for RogueRustSkins 2.2.5 while keeping player permissions, targeting, CUI and execution policy in the plugin.
- Preserved the 4.3.2 Workshop/global asset architecture as the authoritative discovery and cache layer.
- Reserved 4.4.0 as the next stable feature line after successful live development testing.

## 4.3.2

- Moved Steam Workshop published-file and collection resolution into the RogueRust DLL through the new shared IRogueWorkshopService.
- Added keyless Workshop metadata resolution, seven-day cache freshness, batched published-file requests, in-flight request deduplication, collection expansion and persistent reuse through RogueAssets.db.
- Workshop metadata now publishes resolved skins into the shared skin catalogue and supplies preview URLs directly to RogueRust ImageLibrary.
- Added normalized Workshop tag-to-Rust-item lookup so plugins no longer need to repeatedly scan every item definition for every imported skin.
- Kept player-facing import commands, permissions, blacklists and execution policy in consuming plugins; the DLL owns discovery/cache infrastructure only.
- Extended the global asset layer with Steam skin-pack membership based on Rust UnlockedViaSteamItem relationships, while leaving ownership/entitlement decisions to the execution plugin.
- Reviewed external skin/building implementations for architectural gaps. Building grades, wallpaper sides/packs, redirects, searchable catalogue metadata, favourites/sets and player execution policy remain separated so future shared metadata can be added without moving gameplay policy into the DLL.

## 4.3.1-testing

### Global asset/workshop cache migration candidate
- Added the shared RogueRust asset store backed by `RogueRust/Assets/RogueAssets.db` with versioned SQLite migrations.
- Added global Workshop item and collection metadata caching for reuse across RogueRust plugins without requiring a Steam Web API key.
- Added compatibility migration of legacy RogueRust ImageLibrary metadata while leaving legacy data intact for safe rollback during testing.
- Added shared `assets` and `workshop-cache` capabilities while preserving existing ImageLibrary and skin APIs.
- Prepared content cache directories for image and Workshop assets so future consumers can share durable data instead of duplicating plugin-specific caches.
- Added first-class Steam skin-pack indexing from Rust `UnlockedViaSteamItem` relationships, persisted in the global asset database with content/workshop membership for reuse by skin consumers.
- Kept entitlement/ownership policy separate from catalogue discovery so pack/DLC metadata can be indexed globally without forcing plugins to expose paid content.
- Kept Workshop discovery keyless and bounded: known IDs and collections can enrich the shared cache without brittle Steam Community HTML scraping.


## 4.3.0

### RRAM administration/UI release
- Promoted the validated 4.2.9 RRAM testing line to the 4.3.0 stable release identity.
- Finalized the shared RogueRust administration UI foundation, appearance/window primitives and reusable component styling used by RogueRustAdminMenu 2.5.0.
- Retains demand-driven RogueUI rendering, selective refresh policy, shared service reuse and the no-dashboard-polling design.
- Preserves non-blocking immediate actions such as rapid Give without rebuilding the administration workspace.
- Keeps Oxide-first operation and Carbon compatibility through the Oxide compatibility surface.
- Carries forward the validated runtime, diagnostics, GridPower, skin, teleport, vehicle, ImageLibrary and performance infrastructure from the 4.2.x line.

## 4.2.9-testing

### RRAM pre-2.5 polish and performance validation
- Advanced the testing line to 4.2.9 for the final optimisation and UI-polish cycle before the 2.5.0 AdminMenu release.
- Retains demand-driven RogueUI rendering, selective refresh policy, shared service reuse and the no-dashboard-polling design.
- Keeps immediate actions non-blocking and avoids workspace rebuilds for rapid actions such as Give.
- Reserved 4.2.9 for compatibility/performance hardening so the next release line can focus on validated shared UI consumers, including the planned SkinBox conversion.


## 4.2.0-testing

### Completion release from the 4.0.0 baseline
- Consolidated the validated v4.0.0 performance/runtime baseline and subsequent 4.1.x development work into the RogueRust 4.2.0 completion candidate.
- Aligned project, assembly, runtime and plugin-manifest source identity to 4.2.0.
- Retained Oxide-first operation and Carbon compatibility through the Oxide compatibility surface without introducing a hard Carbon dependency.
- Preserved established plugin-facing services and compatibility/API baseline protections.

### GridPower / public power-pole backend
- Added the shared GridPower backend developed through the 4.1.x testing cycle for public power-pole discovery and managed grid state.
- Supports the proven power-pole/streetlight workflow used by RogueRustGridPower without requiring each plugin to duplicate world discovery/state infrastructure.
- Retained bounded discovery/state handling and diagnostics suitable for server-side validation.
- Removed the experimental substation/splitter service implementation from the 4.2.0 release line after testing showed the power-pole-only design to be the safer completed scope.
- Preserved the former experimental substation implementation separately on `testing-substation-backup` for future investigation; it is not part of 4.2.0.

### Performance and runtime hardening retained from v4.0.0
- Retained reduced scheduler and workload hot-path allocations without changing scheduling semantics.
- Retained cached stable Rust world reflection metadata and per-runtime-type entity member lookups to reduce repeated world-query reflection overhead.
- Retained reduced network subscription cleanup/snapshot allocations and HTTP diagnostics allocation overhead.
- Retained bounded HTTP host-interval handling and runtime User-Agent synchronization through the release pipeline.
- Retained cached ADO.NET provider resolution and reduced database diagnostics snapshot allocations while preserving SQLite/MySQL concurrency, retry and circuit-breaker behaviour.
- Retained RogueUI lifecycle/metrics optimisations while preserving canonical full-document rendering for structural changes and safe selective leaf updates.
- Retained drain-first shutdown, workload pressure controls, resilience diagnostics and existing security/integrity helpers.

### Shared framework improvements carried into 4.2.0
- Retained the shared skin catalogue/application/cache/session platform introduced before the v4 baseline.
- Retained embedded binary PNG/FileStorage delivery with validation, fingerprinting, deduplication and no runtime external image-host requirement for packaged assets.
- Retained shared teleport, vanish, vehicle catalogue, F1 administration, world-query, database, HTTP, scheduling, profiling and integration service families.
- Continued the shared-service approach so plugins can avoid duplicating expensive scans, timers, persistence loops and infrastructure.

### Build, CI and release safety
- Updated the source release identity to 4.2.0 so the `testing` branch resolves to `4.2.0-testing` and stable promotion resolves to `4.2.0`.
- CI validates that project, assembly and runtime identities agree before building the protected release package.
- CI continues to require current Rust/Oxide reference assemblies and verifies protected release outputs before artifact upload.
- Public publication remains explicitly controlled: testing and normal CI validation do not overwrite the stable updater channel.
- Retained the unified `RogueRust-Windows.cmd` validation/build/pre-stable/template/protected/package/reference/SDK workflow.
- Retained generated SDK/API reference validation and compatibility-first source-contract gating.

### Release test scope
- Validate clean Oxide startup and shutdown/restart lifecycle.
- Validate Carbon loading through its Oxide compatibility surface.
- Validate the RogueRust plugin family against the 4.2.0 DLL.
- Validate GridPower power-pole discovery, managed power state and streetlight behaviour.
- Validate diagnostics/console commands, updater channel isolation and protected CI artifacts.
- Substation/splitter spawning, preview and access-placement behaviour are explicitly outside the 4.2.0 release scope.

## 4.1.26-testing (experimental, superseded by 4.2.0)
- Added non-spawning nearest-substation splitter preview support for GridPower SAFE-TEST validation.
- Real loaded control-hut door/entry hierarchy was mandatory; inferred-door fallback placements remained unresolved.
- Preview path created no entities and used a minimal visualization path.
- Automatic splitter population remained disabled by default and opt-in only.
- This experimental substation line was removed from the 4.2.0 release scope and preserved on `testing-substation-backup`.

## 4.1.25-testing (experimental, superseded by 4.2.0)
- Hardened GridPower substation splitter SAFE-TEST lifecycle and saved splitter-bank adoption.
- Added placement-delta adoption fallback for stale network IDs and tightened control-hut door resolution.
- Automatic substation splitter population remained disabled by default and opt-in only.
- Superseded by the 4.2.0 power-pole-only completion scope.

## 4.0.0-testing

### Performance and runtime hardening
- Advanced the source, assembly and runtime identity to 4.0.0 while preserving the established plugin-facing API baseline.
- Reduced scheduler and workload hot-path allocations without changing scheduling semantics.
- Cached stable Rust world reflection metadata and per-runtime-type entity member lookups to reduce repeated world-query reflection overhead.
- Reduced network subscription cleanup/snapshot allocations and HTTP diagnostics allocation overhead.
- Tightened HTTP host-interval handling and corrected the runtime HTTP User-Agent to RogueRust/4.0.0.
- Cached resolved ADO.NET provider types and reduced database diagnostics snapshot allocations while retaining existing SQLite/MySQL concurrency, retry and circuit-breaker behaviour.
- Reduced RogueUI lifecycle/metrics overhead while preserving canonical full-document rendering for structural changes and safe selective leaf updates.

### Build, validation and release safety
- Added the unified RogueRust Windows launcher with clean, build, validation, pre-stable, template, protected-build, package, reference-update and SDK-reference actions.
- Kept PowerShell execution-policy bypass process-scoped only.
- Corrected API-baseline export exit handling and retained compatibility-first source-contract validation.
- Changed normal testing/main CI pushes to validate and package only; public publishing requires explicit approval.
- Retained isolated stable/testing updater channels and protected release packaging.

### Compatibility
- v4 is an overhaul rather than a rewrite: existing source files and established services remain in place.
- Oxide remains the primary runtime with Carbon supported through its Oxide compatibility surface and no hard Carbon dependency.
- The supplied RogueRust plugin family remains the regression-consumer set for v4 validation.

## 3.3.11-testing
- Added the shared skin catalogue, application, effective-skin cache and bounded skin-session platform.
- Added persistent skin manifest support and late Steam inventory-definition discovery without a compile-time Steamworks dependency.
- Registered skin services through RogueServices/capability discovery while guarding redirect replacement until state-preserving replacement is validated.

## 3.3.8-testing
- Reworked packaged artwork around binary `.png` assembly resources.
- Added PNG chunk/CRC/IEND validation and immediate FileStorage round-trip verification.
- Retained SHA-256 content deduplication, owner namespaces, stale CRC cleanup and persisted metadata.

## 3.3.7-testing
- Added embedded-asset fingerprinting, stale cache invalidation and explicit delivery diagnostics.

## 3.3.6-testing
- Added the dedicated embedded RogueRust icon catalogue and local-only ImageLibrary asset resolver.

## 3.3.5-testing
- Expanded shared vehicle catalogue metadata and continued F1 administration/ImageLibrary hardening.

## 3.3.0-testing
- Began the coordinated administration-platform overhaul with shared vanish state, RogueUI design/font primitives and shared teleport direction.

## 3.0.0
- Consolidated the mature 2.x framework into the stable 3.x platform with diagnostics, lifecycle safety, API compatibility checks and public documentation.

## 2.x development summary
- 2.9.0: bounded stability/soak monitoring.
- 2.8.0: developer platform, capability contracts and API-baseline release gating.
- 2.7.0: resilience/circuit breakers and drain-first shutdown.
- 2.6.0: ImageLibrary reverse indexes, owner namespaces/pressure and warmup.
- 2.5.0: UI/data efficiency and reusable DB command templates.
- 2.4.0: adaptive observability and recent metrics.
- 2.3.0: pressure attribution, HTTP protection and image content dedupe.
- 2.2.0: runtime pressure/lifecycle hardening.
- 2.1.0: performance/public-package consolidation.
