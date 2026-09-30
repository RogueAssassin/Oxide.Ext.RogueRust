# RogueRust 4.4.0 Features

RogueRust 4.4.0 is the stable global-asset, Workshop, skin-catalogue and shared-service evolution of the RogueRust 4.x framework for Oxide-first / Carbon-compatible Rust servers.

## 4.4.0 release baseline

- Retains the v4 compatibility/performance baseline, RRAM administration framework and GridPower backend.
- Adds a persistent global asset store and shared Workshop service so plugins can reuse metadata rather than duplicate Steam/cache infrastructure.
- Adds shared Workshop collection and Steam skin-pack metadata while keeping player entitlement and execution policy plugin-owned.
- Expands the shared skin catalogue with bounded search and building-grade skin metadata.
- Improves held-item/entity skin refresh through the shared application service.
- Keeps Oxide as the primary runtime and Carbon support through its Oxide compatibility surface without a hard Carbon dependency.
- Maintains protected deterministic release packaging, API compatibility validation, SDK generation and isolated stable/testing update channels.

## Global assets and Workshop

- Persistent global asset database at `RogueRust/Assets/RogueAssets.db` with versioned SQLite migrations.
- Shared asset, Workshop item, collection and Steam skin-pack records through `IRogueAssetStoreService`.
- Keyless known-ID Workshop metadata and collection resolution through `IRogueWorkshopService`.
- Seven-day Workshop metadata freshness by default, bounded batches and in-flight request deduplication.
- Persistent Workshop metadata reuse and ImageLibrary preview URL integration.
- Steam pack membership indexing from Rust `UnlockedViaSteamItem` relationships.
- Discovery/catalogue metadata is shared; ownership, permissions and gameplay policy remain plugin-owned.

## Skin platform

- Shared immutable-snapshot skin catalogue with generation tracking.
- Validated skin application service for safe in-place skins.
- Generation-aware per-player effective-skin cache for permission/ownership filtering.
- Lightweight per-player browser sessions with bounded 48-slot paging and cleanup primitives.
- Runtime imported skin registration/removal with owner/source metadata for Workshop and custom-skin workflows.
- Bounded catalogue search by name, Workshop ID and content ID.
- Shared building-skin catalogue by Rust building grade.
- Held-entity skin/network refresh when safe in-place item skins are applied.
- Redirect skins remain explicitly guarded until state-preserving replacement is implemented and server-tested.

## Framework and SDK
- Strongly typed services plus conventional `RogueRust_*` hook bridges.
- Plugin manifests, dependency validation, versioned capability contracts and generated SDK validation.
- Shared teleport, vanish and vehicle-catalogue contracts for administration plugins.
- Shared GridPower service for safe public power-pole discovery, state management and plugin consumption.

## RogueUI F1 administration
- Document-based UI composition with stable callback tokens and selective updates.
- Full-screen 16:9 F1 workspace geometry with bounded proportional scaling.
- Compact top navigation/category rails and reusable search/grid/tile/table/action/status models.
- Charcoal/slate surfaces, muted olive selection and restrained orange interaction accents.
- Semantic Regular, Medium, Bold, Mono, Brand and Icons font roles.
- Bounded catalogue filtering for item/vehicle browsers.
- Standard item actions: 1, 100, 1000 and native stack.
- No F1-layer polling, timers or per-frame work.

## Vehicle catalogue
- Exact Rust-native item/prefab metadata on vehicle descriptors (`ItemShortname`, `ItemId`, `Prefab`).
- Separate 2/3/4-module car and chassis entries, solo/duo submarines and motorbike-with-sidecar.
- `IRogueVehicleService` and `RogueVehicleService` shared catalogue.
- Animals, Bikes, Boats, Cars, Helicopters, Misc, Siege and Trains categories.
- Stable IDs, display names, image keys and search text.
- Bounded category/search filtering.
- Spawn/entity creation intentionally remains outside the catalogue until safe spawn resolution is finalized.

## Image/Asset Pipeline
- Binary `.png` assembly resources remain the canonical packaged-artwork format.
- Embedded PNGs are signature/dimension/chunk-CRC/IEND validated and stored directly, preserving the exact packaged artwork.
- New embedded FileStorage writes are immediately read back and verified before plugins receive the CRC.
- Unity ImageConversion remains available only for non-PNG normalization.
- Embedded assets are content-fingerprinted and stale FileStorage records are automatically refreshed when packaged artwork changes.
- Dedicated assembly-embedded `Icons/` asset catalogue for RogueRust plugin branding.
- No runtime HTTP, GitHub or SteamID dependency is required for embedded artwork.
- URL download and Rust FileStorage persistence with SHA-256 content deduplication and shared CRC reuse.
- Five-minute failed-request cooldown suppresses repeated broken URL downloads.

## Teleport and vanish administration
- Safe-position options and destination resolution for position/grid/monument workflows.
- Previous-location tracking and Back support.
- Explicit teleport success/failure results.
- Lightweight owner-scoped shared vanish state with capability discovery.
- Rust-specific vanish networking/NPC/gameplay policy remains plugin-owned and event-driven.

## Database / HTTP / runtime
- SQLite and MySQL/MariaDB-compatible providers, migrations, transactions, batches and bounded concurrency.
- Bounded HTTP responses/concurrency, retries, attribution and circuit breakers.
- Scheduler/workload helpers, cooldowns, pooling, events, logging, profiling and soak monitoring.
- Atomic/debounced data and configuration persistence.

## World / resilience / security
- Terrain, topology, grid, monuments, roads, rivers, rail, entity/spatial and spawn helpers.
- Resource-pressure attribution, bounded stability history and drain-first shutdown.
- Secure UI callbacks, integrity/security diagnostics and permission helpers.
- Discord webhook and typed integration adapter surfaces.

## v4.0.0 -> v4.4.0 hardening
- Carries forward reduced allocation pressure in scheduler, workload, network, HTTP, database and RogueUI diagnostic/lifecycle paths.
- Retains cached stable world/reflection and database-provider metadata while continuing to read live server/entity state.
- Preserves bounded HTTP/database concurrency, retry, circuit-breaker and drain-first shutdown behaviour.
- Extends the shared server-service model with the GridPower power-pole backend and the 4.4.0 global asset/Workshop infrastructure.
- Removes the experimental substation/splitter services from the release line after testing showed the power-pole-only model to be the safer completed scope.
- Keeps unified Windows clean/build/validate/pre-stable/template/protected/package/reference/SDK tooling behind `RogueRust-Windows.cmd`.
- CI validates version identity and packages protected artifacts; public publication remains explicitly controlled.
- Existing public API contracts remain guarded by the source API baseline.

## Release channels
Stable 4.4.0 uses `update-manifest.json`. Testing 4.4.0 builds use `update-manifest-testing.json`; testing publication never overwrites the stable updater pointer. Public stable publication is produced only from a protected release artifact.