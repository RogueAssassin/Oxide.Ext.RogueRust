# Changelog

## 2.1.0 — Performance and public-release hardening

- Promoted the 2.0.2 codebase to the 2.1.0 framework baseline without removing existing public ImageLibrary, UI, command, database, scheduler or compatibility APIs.
- Changed native ImageLibrary lookups so a FileStorage CRC is validated once per runtime/community entity and subsequent reads are dictionary-only instead of reloading PNG bytes on every UI lookup.
- Added validated-CRC invalidation when images are replaced/removed, metadata reloads, FileStorage entries become stale, or the Rust CommunityEntity changes.
- Added 8 MiB remote source-size, 4096x4096 decoded-dimension and 3 MiB normalized-PNG safety limits to prevent image-cache memory spikes.
- Cached successful Unity ImageConversion method discovery instead of repeating reflection lookup for every downloaded image.
- Changed ImageLibrary metadata persistence to a true locked snapshot plus temporary-file/replace write strategy.
- Coalesced normal ImageLibrary metadata mutations through a short delayed save instead of rewriting `images.json` after every downloaded image.
- Kept image decoding/download processing sequential by default to avoid concurrent Texture2D/decode memory pressure on the game server.
- Added a global HTTP execution gate limiting blocking HTTP attempts to eight concurrent workers while preserving cancellation, retries and per-host pacing.
- Changed ADO/MySQL `ExecuteBatchAsync` to run the complete transaction/batch through one worker hop rather than one `Task.Run` per SQL statement.
- Added `RogueImageLibraryExample` and `RogueDatabaseBatchExample` samples.
- Added automated `Oxide.Ext.RogueRust-v2.1.0-samples.zip` generation/checksum and public GitHub release upload.
- Consolidated public Markdown documentation to `README.md`, `CHANGELOG.md` and `FEATURES.md`; historical fix/migration Markdown files are folded into the current docs.
- Updated public documentation with server-owner installation/support guidance, ImageLibrary SteamID clarification, current hooks/commands, performance guidance and links to the samples package.

## 2.0.x — Native ImageLibrary 2.x baseline

- Unified framework/runtime/assembly/updater/sample/template/workflow versioning on the 2.x line.
- Added native `IRogueImageLibraryService` backed by Rust FileStorage for CUI image caching without requiring the standalone ImageLibrary plugin.
- Added queued remote downloads, persistent URL/CRC metadata, CommunityEntity/wipe invalidation and stale-entry recovery.
- Added RogueUI `ItemIcon(itemId, skinId)` support for native item/skin previews.
- Added required UnityWebRequest/Rust.Data build-reference synchronization.
- Removed the compile-time UnityEngine.ImageConversionModule dependency that conflicted with net48/netstandard reference versions.
- Resolved Unity `ImageConversion.LoadImage` and `EncodeToPNG` dynamically from the running Rust server while retaining PNG validation.

## 1.9.1 — Native images and configuration layout

- Introduced the native RogueRust ImageLibrary service and image-related capabilities.
- Moved RogueRust/plugin configuration toward the direct host config directory layout with migration from legacy nested layouts.
- Added native Rust item/skin icon rendering support.

## 1.9.0 — Native command lifecycle

- Removed framework dependence on host `AddChatCommand`, `AddConsoleCommand`, `RemoveChatCommand` and `RemoveConsoleCommand` overloads.
- Routed RogueRust chat/RCON command execution through native Rust hook paths and the Rogue command service.
- Added `roguerust.exec <command> [args...]` as the local console/RCON RogueRust command gateway.
- Kept unknown commands flowing to the normal Rust/Oxide/Carbon command pipeline.
- Hardened world-size resolution for Carbon/Mono compatibility.

## 1.8.0 — Stable

- Promoted the live-tested 1.7.5 runtime, callback and RogueUI reconciliation core to the official stable baseline.
- Added `WindowHeader` with an integrated top-right close control and optional subtitle.
- Added `CloseButtonTopRight`, `Toast`, `ConfirmationModal` and `Pagination` high-level RogueUI helpers using validated Oxide CUI primitives.
- Added `RogueUiLayout.Inset` and `ResponsiveCentered` layout helpers.
- Kept structural/navigation transitions atomic and restricted partial replacement to safe isolated leaf changes.
- Preserved stable keyed server-side UI actions for tabs, pages, toggles and finite-state controls.
- Kept Oxide/uMod as the canonical runtime/API contract with Carbon supported through its Oxide-compatible layer.
- Synchronized extension, assembly, updater, HTTP and readiness version metadata to 1.8.0.
- Validated the 1.8.0 line on a clean server with RogueUI open/close, navigation and repeated interaction testing.

## 1.7.x — RogueUI reconciliation and runtime compatibility

- Added per-element fingerprints, dirty tracking, render/skip/destroy telemetry and reusable `RogueUiTemplate` composition.
- Added modal/navigation/page-indicator helpers and callback argument support.
- Corrected dual-runtime command registration so RogueRust does not hard-bind to a single Oxide command-library CLR overload.
- Reworked unsafe generic `update:true` writes after live testing exposed Rust client `AddUI` failures.
- Aligned button serialization with Oxide's canonical button + child-text structure.
- Corrected input-field wire names and culture-invariant CUI anchor formatting.
- Changed reconciliation to favour client safety: identical documents are skipped, one safe leaf may be selectively replaced, and structural/multi-element changes rebuild atomically.
- Added stable server-side action callbacks for navigation so tabs/pages no longer depend on client-supplied state arguments.
- Expanded the regression dashboard with genuinely different Overview and Metrics trees.
- Corrected internal framework version identity that had remained on 1.6.0 during part of the 1.7 development cycle.

## 1.6.0 — RogueUI state, layouts and telemetry

- Added stable document fingerprints and payload-length tracking for lower-memory unchanged checks.
- Added unchanged suppression to `ShowIfChanged` and `Update`.
- Added UI byte/timing telemetry plus active document, callback and state counters.
- Added expiring player-bound callbacks while preserving the original callback API.
- Added owner/player-scoped transient UI state with automatic unload/disconnect cleanup.
- Added grid, vertical and horizontal layout helpers.
- Added badge, toggle, tab, spacer and text-input controls.
- Expanded theme tokens for success, warning and muted surfaces.

## 1.5.0 — RogueUI performance and database hardening

- Added cached RogueUI serialization, incremental update foundations, unchanged-render suppression and reusable UI components.
- Added secure player-bound UI callbacks.
- Improved local SQLite persistence with a guarded long-lived connection and WAL-oriented pragmas.
- Corrected MySQL/MariaDB connector discovery to support the bundled MySqlConnector implementation.
- Formalized the Oxide-first architecture with Carbon as a supported secondary runtime through compatibility.

## 1.4.0 — Workload coordination

- Added owner-scoped throttling, debouncing, next-tick coalescing and unique repeating jobs.
- Reduced event-dispatch allocations and added workload capability discovery.
- Added performance guidance for replacing duplicate plugin timers and repeated hot-hook work with shared RogueRust services.
- Kept 1.3 APIs source-compatible; workload coordination was additive and opt-in.

## 1.3.0 — Consolidated stable framework

- Promoted the 1.2 release-candidate work into a stable framework baseline.
- Revalidated managed references against the contemporary Rust/Oxide server build used for the release.
- Hardened Windows PowerShell signing/release verification.
- Removed hard-coded release-version propagation from build tooling.
- Updated plugin templates/SDK defaults and repaired documentation/metadata encoding issues.

## 1.2.x — Service expansion and release hardening

- Added world/entity/terrain/monument/topology/spawn services.
- Added process/CLR runtime metrics, hotspot detection and optimisation recommendations.
- Added cron/absolute scheduling and bounded internal message transport with compression, chunking and rate limiting.
- Added plugin manifests, dependency validation and generated command help/usage.
- Added artifact integrity/security services and developer service exploration.
- Added pathfinding, terrain analysis, raw map rendering, image caching, weighted loot selection, entity snapshots, binary serialization, font metadata and richer Discord components.
- Consolidated documentation and introduced Windows GitHub Actions/public-release publishing.
- Rebuilt the release pipeline around protected-by-default builds, canonical version normalization, manifest/signature verification and protected assembly checks.

## 1.1.0 — Shared utility/service layer

- Established shared pooling, serialization, utility, typed-data and Discord services.

## 1.0.x — Initial 1.x foundation

- Established the original RogueRust 1.x extension/framework line that later releases expanded into the current shared service and plugin SDK platform.
