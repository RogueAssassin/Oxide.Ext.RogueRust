# RogueRust 2.1.0 Features

RogueRust is an Oxide-first extension framework for Rust with CarbonMod compatibility through the Oxide-compatible runtime surface. It is intended to be the shared infrastructure layer used by performance-conscious Rust plugins.

## Core framework

- Service registry, lifecycle ownership and capability discovery.
- Plugin manifests, dependency checks and compatibility helpers.
- Owner-scoped cleanup when plugins unload.
- Shared logging, profiling, runtime metrics and circuit breakers.
- Pooling, serialization, binary serialization and utility services.

## RogueUI

- Rust/Oxide CUI document builder.
- Stable player-bound server-side callbacks/actions.
- Per-player transient UI state.
- Unchanged-document suppression to avoid duplicate AddUI traffic.
- Conservative safe leaf replacement and atomic structural rebuilds.
- Panels, surfaces, labels, buttons, images, item icons, progress bars, badges, toggles, tabs and text input.
- Windows/headers, top-right close controls, notifications/toasts, confirmation modals and pagination.
- Responsive/grid/vertical/horizontal layout helpers.
- UI render, callback, payload and reconciliation telemetry.

## Native ImageLibrary

- Built-in remote image URL registration and download queue.
- Rust FileStorage-backed PNG CRC cache; no standalone ImageLibrary plugin is required.
- No player SteamID or Steam Web API key is required by the core image cache.
- Optional `imageId` value is a cache variant/skin/workshop identifier, not player identity.
- Lazy FileStorage validation: an image is validated once per runtime/community entity, then normal reads are dictionary-only.
- Stale FileStorage entries self-heal and are queued for re-download when a source URL is known.
- 8 MiB source-download limit, 4096x4096 decoded-dimension limit and 3 MiB normalized-PNG limit.
- Cached Unity ImageConversion method discovery.
- Debounced, snapshot-safe and atomic metadata persistence.
- Native RogueUI `ItemIcon(itemId, skinId)` support for cheap Rust-resolved item/skin previews.

## Commands

- `[RogueCommand]` metadata with aliases, descriptions, usage, permissions and cooldowns.
- Argument binding and validation helpers.
- Native Rust hook dispatch rather than hard binding to host `AddChatCommand`/`AddConsoleCommand` overloads.
- Chat command interception through Rust hooks and RCON/server execution through the Rogue command gateway.
- `roguerust.exec <command> [args...]` for local console/RCON execution.

## Data and databases

- Typed configuration and data-file helpers.
- Shared TTL cache.
- Rogue-owned SQLite support for local persistence.
- Optional MySQL/MariaDB provider for shared/multi-server data.
- Schema migrations and parameterized commands.
- Transaction-backed `ExecuteBatchAsync`.
- In 2.1.0, ADO batches use one worker hop per batch instead of one ThreadPool hop per statement.

## Performance coordination

- Throttle repeated hot-hook work.
- Debounce rapid saves/refreshes.
- Coalesce duplicate next-tick work.
- Unique repeating jobs so multiple code paths do not create duplicate timers.
- Runtime profiler and slow-operation counters.
- Bounded HTTP execution in 2.1.0: at most eight blocking HTTP attempts execute concurrently across the framework.

## Scheduling

- Delayed and repeating jobs.
- Absolute/cron scheduling where supported.
- Owner-scoped cancellation and unload cleanup.
- Scheduler diagnostics.

## HTTP, messaging and integrations

- Async HTTP requests with retry policies, cancellation and per-host pacing.
- Global bounded execution to prevent plugin request bursts from saturating the CLR ThreadPool.
- Discord webhook support.
- Internal transport/message services.
- Adapter registry and optional economy integrations.

## World and gameplay helpers

- Player lookup and nearby-player queries.
- Map grid conversion.
- Terrain, topology, monument and spawn services.
- Entity queries/snapshots/serialization.
- Pathfinding and terrain analysis.
- Map rendering, image cache, loot selection and font metadata utilities.

## Security and release system

- Artifact integrity service.
- Protected release pipeline.
- Release manifests and SHA-256 checksums.
- Update manifest generation and update checks.
- GitHub workflow support for protected public releases.
- Separate downloadable Samples package generated for each 2.1.x release.
