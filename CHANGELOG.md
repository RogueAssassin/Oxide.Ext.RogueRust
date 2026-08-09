# Changelog

This changelog consolidates the RogueRust 1.x development line into one public history. Patch releases used to validate/correct the 1.7 UI transition are grouped under 1.7.x so the public history remains useful rather than repetitive.

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
