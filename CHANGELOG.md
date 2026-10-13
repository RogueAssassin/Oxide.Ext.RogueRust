## 4.4.63 — Building service

- **New service `Rogue.Buildings`** (`IRogueBuildingService`): finds the building a player means (the block they are looking at, otherwise their tool cupboard's), checks cupboard authorisation, collects a building's blocks, works out what a grade change costs using the game's own prices, takes that cost all-or-nothing and reports what is missing, gives refunds, and applies grade, skin and colour changes through the game's own call. Plugins queue the per-block work with `Workloads.Batch`.
- **`RogueBuildingGrades`:** grade numbers, display names, parsing of names such as `stone`, `hqm` and `armored`, and cost rounding.
- **Tests:** 105 unit tests, adding grade parsing, stepping and cost rounding.
- Paired plugin updates (separate files): the new RogueRustBuildingManager 1.0.0, RogueRustSkins 2.3.2 (hands structure skinning to it) and RogueRustServerInfo 1.1.2.

## 4.4.62 — Plugin config and data attributes, DeathNotes names, HeliCrash icon

- **Fix (config and data files):** `[JsonProperty]` and `[JsonIgnore]` on plugin classes are now honoured by `LoadConfiguration`/`SaveConfiguration` and `LoadData`/`SaveData`. Oxide compiles plugins against its own merged copy of Json.NET, so the extension never saw those attributes: files were written with member names, a setting typed under its documented name (for example `"Disable Vanilla Spawns": true` in RogueRustEventDirector) was silently discarded on the next load, and ignored members were written out. Existing files still load, because a renamed member also accepts its old member-name key; they are rewritten with the declared names on the next save. Once rewritten, a file is not readable by 4.4.60 or earlier.
- **Config files:** saved lists now replace a class's built-in defaults instead of being appended to them.
- **Icons:** `Icons/RogueRustHeliCrash.png` is embedded for the RogueRustHeliCrash on-screen alert.
- **Dictionaries:** added `DeathNotesNames` (display names for the killers and victims the original Death Notes shipped with: animals, NPCs, traps, turrets, vehicles), `DeathNotesWeaponNames` and `DeathNotesAttachments`.
- **Dictionaries:** `DeathNotesWeapons` gains about 85 prefab aliases: held tools and guns, RPG, MLRS, mortar, cannon and siege projectiles, molotov, bee and flashbang grenades, modular cars, bikes and snowmobiles.
- **Tests:** every embedded dictionary is loaded and checked, and the attribute handling is covered. 82 unit tests.
- Paired plugin updates (separate files): RogueRustDeathNotes 2.5.3, RogueRustEventDirector 1.3.2, RogueRustHeliLeashControl 2.1.3, RogueRustServerInfo 1.1.1, RogueRustFurnaceSplitter 2.1.3, RogueRustHackableCrateTracker 2.1.1, and the new RogueRustHeliCrash 1.0.0. 4.4.61 was a test build and was not released.

## 4.4.60 — Stability release

- **Plugin unload:** each cleanup step (events, network, image callbacks, cache, commands, UI, workloads, scheduler, HTTP, database) is isolated, so a failure in one can no longer leave an unloaded plugin's timers, requests or UI alive in the others.
- **HTTP:** a POST or PATCH is no longer replayed after a timeout or 5xx unless the caller supplies a retry policy, because the server may already have acted on it (for example a webhook message sent twice). 429 responses still retry for every method. A date-form `Retry-After` is capped at five minutes like the seconds form.
- **Database (MySQL and custom providers):** argument errors, a missing provider and unregistered connections are not retried and do not count against the circuit breaker.
- **Events:** removed a race where a subscription added while its event group was being emptied could be lost.
- **Updater:** the "check in progress" flag is always cleared, so one failed check cannot disable later ones.
- **Caches:** expired entries in the image byte cache are swept with the main cache.
- **SDK:** `SaveDataInBackground(key, value)` on `RogueRustPlugin` snapshots on the game thread and writes on a worker thread, for periodic and dirty-flag saves.
- **Tests:** 68 unit tests, adding events, the in-process network bus and HTTP retry decisions.

## 4.4.59 — Shared scheduler timer, indexed entity queries, sliced pathfinding

- **Scheduler:** every job is now driven by one shared timer and a due-time heap instead of one system timer per job. Cancelling or rescheduling no longer allocates or disposes timers. Behaviour is unchanged; ordering, cancellation, repeat counts, rescheduling and 2,000 concurrent jobs are covered by tests.
- **World:** `World.FindNearby(position, radius, typeName, maxResults)` (and `RogueWorldService.FindEntitiesIndexed`) query Rust's own server entity grid, so the cost follows the entities near the position rather than the size of the world. It falls back to the full scan if the grid is unavailable or the candidate buffer overflows. `FindEntities` is unchanged.
- **Pathfinding:** `Pathfinding.FindPathSliced(start, goal, completed, options, nodesPerTick)` runs the same search a slice per tick and reports the result through a callback. `FindPath` is unchanged and now shares the same resumable search.
- **Source:** the minified service files (GridPower, circuit breakers, network, vehicles, compatibility, advanced services, image library) were re-laid out one statement per line by a syntax-preserving tool; their token streams are identical before and after.

## 4.4.58 — Unit tests, image index saves and release hardening

- **Tests:** a unit-test project (`tests/`) covers version comparison, cron parsing, the scheduler (including a repeating job surviving a failure), the cache, cooldowns, circuit breakers, pathfinding, PNG validation, the topology masks and the update-manifest signing format. CI runs it before every build.
- **Performance:** the ImageLibrary index is serialized once, without an intermediate copy, and the debounced save writes the file on a worker thread. The index is flushed at shutdown.
- **Fix:** logging a scheduled-job failure can no longer affect the job itself.
- **Publishing:** re-running the pipeline for a version that is already public keeps that version's binaries and updater pointer and only refreshes notes and the changelog. The update manifest is written without a byte-order mark.
- **Docs:** the README and feature list describe the current release flow.

## 4.4.57 — Signed updates, single version source, fixes and performance

- **Updater:** the release public key is now built into the DLL and a valid manifest signature is required before an update is installed. Unsigned or wrongly signed releases are refused.
- **Versioning:** the version is declared once, in the `VERSION` file. Project metadata, assembly attributes, `RogueRustExtension.InformationalVersion`/`CurrentVersion`, the Windows tooling and both GitHub workflows read it from there; nothing rewrites source files any more.
- **Fix:** `RogueTopologyService` road, river and rail masks now match Rust's `TerrainTopology` values.
- **Fix:** the MySQL provider resolves `MySqlConnector` from `Oxide.MySql.dll`.
- **Fix:** unregistering a database (including on plugin unload) no longer closes a SQLite handle or disposes a gate under a running operation.
- **Fix:** only transient SQLite errors are retried and counted against the circuit breaker.
- **Fix:** plugin unload now removes network subscriptions and pending image callbacks.
- **Fix:** command arguments are validated before the cooldown starts; server console and RCON can run permissioned commands.
- **Fix:** a repeating or cron job that throws keeps its schedule, and job failures are logged.
- **Change:** `RogueRustPlugin` cache and cooldown helpers prefix keys with the plugin name.
- **Performance:** terrain, topology, world-size and entity queries call the game API directly instead of through reflection; pathfinding uses a binary heap; UI callback expiry sweeps are throttled and CUI calls use cached delegates; PNG validation uses a table-driven CRC; expired cache entries are swept every minute.
- **Performance:** `SaveDataDebounced` serializes on the game thread and writes on a worker thread; asset-store database writes are queued instead of blocking the caller; ImageLibrary downloads up to three images at once with one decode per frame.
- **Tooling:** `RogueRust-Windows.cmd` no longer defaults to version 4.4.0, and its Protected build action works again.
- **Publishing:** the public release now contains only the signed protected DLL, its signed manifest and `SHA256SUMS.txt`, and is refused unless the signature verifies. The release page shows only that version's changelog section; the full `CHANGELOG.md` is updated on the public repository for stable releases.

## 4.4.56 — Pole matching fix (testing)

- **Fix:** Rust names each placed map prefab after its full lowercase prefab path (`.../powerline_pole_a.prefab`), so 4.4.55's "name starts with `powerline_pole`" check matched nothing. Poles are now matched anywhere in the path, and the pole model is the prefab file name.
- When no poles are found, the status now reports how many placed prefabs and categories were scanned, the Rust marker count, and up to 8 power-related prefab names seen.

## 4.4.55 — Pole transformers on Rust's roadside poles (testing)

- **Why 4.4.52–4.4.54 fitted nothing:** they looked for `PowergridPowerline` pole entities, which this map does not have. The server log showed 0 pole slots: only 32 map entities exist, and all 164 transformers sit on `PowergridManager` markers.
- **Pole discovery:** poles now come from `World.SpawnedPrefabs`, the world's record of placed map prefabs (`powerline_pole_*`). The list is read once per session, with no scene scan.
- **Mount point:** for each pole model, it is learned from Rust's own `PowerlinePowergridAccessPointSpawn` markers on poles of that model. Rust's transformer prefab is placed at that point on the configured share of the other poles.
  - Marker poles count first.
  - A pole with a transformer within 1.5 m of its mount point is skipped.
  - Pole models with no marker anywhere on the map are left alone.
- The status reports poles found, pole models with a known mount point, Rust markers, poles without a mount point, and the usual fitting counts.

## 4.4.54 — GridPower cleanup, pole discovery diagnostics (testing)

- **Removed legacy grid code:**
  - The stacked-duplicate audit (`AuditNativeAccessPoints`) and the non-vanilla transformer audit/removal (`AuditNonVanillaAccessPoints`, `RogueNativeGridOptions.RemoveNonVanillaAccessPoints` / `RemoveWiredNonVanillaAccessPoints`), with the spawn-point classifier behind them.
  - The obsolete pre-4.4.47 player-pole members (`PlayerPowerPoles`, `PlayerSubstations`, `PlayerReferenceFilePath`, `RefreshPlayerInfrastructure`, `GetPlayerInfrastructureStatus`).
  - The stacked count in the native grid status.
  - GridPower is now only: streetlights, the native grid output policy, and native pole transformers.
- **Native pole transformers:**
  - Pole discovery falls back from the typed scene lookup to Rust's server entity list.
  - The status now reports how many pole entities were found and by which method, how many have no mount slot, and the exact reason when nothing could be fitted. That case is also logged as a warning.
- Pairs with RogueRustGridPower 2.0.0.

## 4.4.53 — ConfuserEx hot-path profile and on-demand hooks (testing)

### ConfuserEx
- Bundled ConfuserEx is 1.6.0 (mkaring), the latest stable release. The tooling check now fails on anything older than 1.6, because 1.4.0 has a known control-flow bug that breaks methods at runtime.
- Hot-path types are excluded from `ctrl flow` and `constants` via `[Obfuscation(Feature = "-ctrl flow;-constants")]`. ConfuserEx's constants runtime re-decodes and allocates a string on every use of a string literal, and control flow adds a dispatch state machine to every call. Excluded types:
  - Profiler, Scheduler and Workloads services
  - Permission service
  - Native power-grid (Harmony prefixes)
  - Spatial index
  - `RogueRustPlugin` base wrappers

  The rest of the DLL keeps both protections. Renaming stays off.
- The protected-assembly check now also requires the Harmony prefix names, `__result` and `RogueDynamicHookAttribute` to survive protection.

### Plugins
- New `[RogueDynamicHook("group")]` for `RogueRustPlugin` hook methods. RogueRust unsubscribes them once the plugin loads. The plugin turns a group on or off with `SetDynamicHooks(group, bool)` / `SetDynamicHooks(bool)` and checks it with `AreDynamicHooksEnabled(group)`. This replaces the hand-written `Subscribe`/`Unsubscribe` bookkeeping in plugins for hooks like `OnPlayerInput` and `OnEntityTakeDamage`.

## 4.4.52 — Transformers on native poles, with density (testing)

- **Fix:** the 4.4.51 legacy audit only recognised `PowergridManager` spawn points and missed the transformer slot on each native pole. Rust's own pole transformers were classed as non-vanilla, and `remove all force` could delete them. Pole slots are now recognised as native placements.
- **Native pole transformers:** Rust's power poles are `PowergridPowerline` entities. Each has its own mount slot (`spawnAccessPointHere`) and prefab (`accessPointPrefab`). On a fresh wipe Rust rolls `ShouldSpawnAccessPoint()` per pole, and about 35% pass.
  - New `ConfigureNativePoleTransformers(enabled, densityPercent, seed, spawnsPerFrame)` fits transformers on exactly that slot, with that prefab, and calls the pole's own `OnAccessPointSpawn`.
  - Density is the share of all native poles that should have a transformer. Rust's own picks count first; the rest follow a stable seed order.
  - A pole that already has a transformer within 1 m of its slot is skipped. Lowering the density never removes any.
  - Spawns are spread across frames (4 per frame by default). They save like Rust's own, so they are not respawned after a restart.
- New `GetNativePoleTransformerStatus()`.
- Pairs with RogueRustGridPower 1.9.9.

## 4.4.51 — Remove transformers RogueRust spawned in older builds (testing)

- Builds up to 4.4.46 spawned extra power-pole transformers (`powergrid_powerline_io.static.prefab`) with saving enabled, so they kept reloading from the server save after 4.4.47 stopped spawning. RogueRust has not spawned access points since 4.4.47, and this release removes the leftovers.
- New `AuditNonVanillaAccessPoints(remove, includeWired, force)`: Rust's PowergridManager only spawns access points at its registered `PowerlinePowergridAccessPointSpawn` map points on a fresh wipe. An access point that is not within 0.5 m of one of those points is classed as non-vanilla and is listed, and removed when asked.
  - Wired ones are kept unless `includeWired`.
  - Removal is refused if Rust's spawn points cannot be read, or if more than half of all access points would be classed as non-vanilla, unless `force`.
- New `RogueNativeGridOptions.RemoveNonVanillaAccessPoints` / `RemoveWiredNonVanillaAccessPoints`: runs that cleanup on the next tick after `ConfigureNativeGrid`.
- Pairs with RogueRustGridPower 1.9.7.

## 4.4.50 — DeathNotes coverage and stable release readiness

- Added combat-type dictionary aliases for shark, travelling-vendor turret, excavator, car shredder, cactus, locomotive, work cart and train; expanded shared entity and weapon labels.
- Aligned `AssemblyInfo.cs`, project metadata, `RogueRustExtension` runtime version and CI release validation with 4.4.50.
- Preserved the 4.4.49 profiler/scheduler allocation improvements and October compatibility changes.
- Updated stable release documentation and protected public-release workflow version identity; stable and testing updater manifests remain separate.

## 4.4.49 — Hot-path allocation cuts for plugins (testing)

- Profiler: `Measure()` no longer builds an `owner/category/operation` key string or allocates a `Stopwatch` on every completed scope. Metrics use a composite struct key and `Stopwatch.GetTimestamp()`. Reported metrics are unchanged.
- Scheduler: zero-delay one-shot jobs (`Delay(TimeSpan.Zero, ...)`, `CoalesceNextTick`, zero-delay `Debounce`) no longer create a `System.Threading.Timer`. They dispatch straight to the existing deferred path (Oxide NextTick or the thread pool), so callbacks stay asynchronous. Repeating and cron jobs keep their timer.
- Paired plugin updates (separate files): AdminMenu 2.5.2, DeathNotes 2.5.1, FuelStackOptimizer 2.1.1, FurnaceSplitter 2.1.1, HeliLeashControl 2.1.1, RandoSpawns 2.1.1, RemovalTool 2.0.2, Skins 2.3.1.

## 4.4.48 — Native grid: power-plant requirement override (testing)

- Added `RogueNativeGridOptions.OverridePowerPlantRequirement`. In `Convars` mode it makes every native access point deliver the configured maximum output (`powergrid.powerlinemaxpoweroutput`) immediately, without power-plant fuses or stages.
- The override reuses the Fixed-mode output patch. It honours `FixedRequiresGridOnline` (follows street-light hours) and `RespectPowergridDisabled`.
- `GetNativeGridStatus` now reports `PowerPlantRequired=YES|NO (override)` and the pinned output.
- Pairs with RogueRustGridPower 1.9.6 ("Override Power Plant Requirement").

## 4.4.47 — GridPower rebuilt on Rust's native power grid (testing)

### Native power grid
- GridPower now works with Rust's own `PowergridIOAccessPoint` entities, which Rust spawns, saves and lets players wire. RogueRust no longer spawns its own transformers.
- New `ConfigureNativeGrid(RogueNativeGridOptions)` output policy:
  - `Vanilla`: no change.
  - `Convars`: sets Rust's `powergrid.powerlinebasepoweroutput` / `powerlinemaxpoweroutput`; output still follows power-plant stages. Original values are restored on `Vanilla` or shutdown.
  - `Fixed`: constant output on every access point via a Harmony override, optionally only while the RogueRust grid is online and respecting `powergrid.enabled`.
- Output changes are re-pushed to access points in frame-budgeted batches (1 ms / 32 per frame by default).
- Access points are indexed with a typed lookup (about 1k objects) instead of an 820k-transform scene scan, which took about 27 s on a 4500 map in 4.4.45.
- New diagnostics: `GetNativeGridStatus`, `InspectNativeAccessPoint` (saving flag, wired slots, energy, max output) and `AuditNativeAccessPoints` (reports stacked access points left by older RogueRust test builds and can remove unwired duplicates only).
- `TryFindNearestPlayerPole` now resolves to the nearest native access point, so ladder placement no longer needs reference JSON.

### Removed
- RogueRust pole discovery, the pole-type catalogue, road review, pole diagnostics files and substation clustering.
- `RogueGridPowerPlayerService`, `RogueGridPowerRecoveryService`, `RogueGridPowerControlledTest`, `RogueGridPowerClimbService` and `RogueGridPowerClimbabilityService`.
- Automatic pole discovery on streetlight refresh (the source of the long startup scan).
- Legacy interface members (`PlayerPowerPoles`, `RefreshPlayerInfrastructure`, `GetPlayerInfrastructureStatus`, `PlayerSubstations`, `PlayerReferenceFilePath`) remain as `[Obsolete]` forwards for one release.

### Unchanged
- Streetlights: discovery, density/seed selection and 4.4.46 streamed spawning.

### Requires in-game verification
- Wired access points keep their wires across a restart.
- `Convars` changes take effect without waiting for a power-plant stage change.
- `Fixed` output powers wired circuits.
- Run the audit once on servers that used earlier RogueRust transformer tests.

## 4.4.46 — Server performance pass: GridPower streaming and hot-path indexing (testing)

### GridPower
- Streetlight spawn/despawn is now streamed across frames (default 1.0 ms / 12 fixtures per frame) and ordered nearest-to-players first, instead of creating or killing every managed light entity in one frame at dusk and dawn. `IRogueGridPowerService.ConfigureStreaming(...)` tunes it; `maxFixturesPerFrame = 0` restores the legacy single-frame behaviour.
- Player-grid population no longer scans every `BaseNetworkable` in the world for each transformer it places. Native power-grid IO positions are indexed once per reconciliation pass.
- The independent-power Harmony prefixes (`GetCurrentEnergy`/`GetHasPower`, which run for every power-grid IO entity on every IO update) now use an O(1) ownership set instead of enumerating every owned transformer.
- Pole discovery uses allocation-free name checks for every scanned transform (power-pole/substation tests no longer lower-case each name). Streetlight exclusion and native IO/access-spawn association use spatial buckets, and road distance plus native proximity are computed once per pole instead of up to two road scans and six native scans per pole.
- Climbability discovery examines only transforms named as pole variants instead of walking ten parents (with a lower-cased copy per hop) for every transform.
- `RemoveAll` is a single pass (was O(n²)). Transformer adoption uses a spatial index. Output refreshes on configure are skipped when the effective output is unchanged.
- `IsStreetlight`/`SetStreetlight` are O(1) lookups.
- Added `TryFindNearestPlayerPole(...)` so plugins can do pole proximity checks without re-reading the reference JSON, and `PendingStreetlightUpdates` plus streaming metrics in `GetStreetlightStatus()`.

### Core
- Console command lookup no longer allocates a lower-cased copy for every console command (`OnServerCommand` hot path).
- `OnEntityKill` skips world-event bookkeeping for entity types that are never tracked.
- `IRogueWorldService.FindEntities` reads component transforms directly instead of using reflection per entity.
- Workshop resolution publishes skins once per Steam batch (`RegisterRange`) instead of rebuilding the whole skin catalogue per item.
- The full skin-catalogue SQLite rewrite runs on the database worker instead of blocking the caller.
- Steam skin-pack indexing at startup writes only new or changed packs in one transaction (previously they were never persisted because indexing ran before the store was ready). The legacy ImageLibrary import is a single transaction.
- Repeating scheduler jobs re-arm their existing timer instead of allocating a new one each run.
- Builds on the 4.4.45 schema-6 / pole-filter-12 discovery and roadside eligibility rules without changing reference formats or classification.
- In-game verification required: dusk/dawn should log the new "applied" line with `MaxFrameMs`; transformer population, adoption after restart and pole counts should match 4.4.45.

## 4.4.43 — Transformer activation and in-game validation (testing)

- Populate schema-6 player pole references from explicitly enabled roadside-priority structural families only when validated within 12 m of a discovered road spline.
- Exclude poles associated with native power-grid IO/access spawn points; preserve vanilla grid infrastructure and do not enable all discovered poles automatically.
- Align the player transformer population manager, single-entity controlled test, and recovery reference validation to schema 6.
- Reject destroyed/non-networked entities after spawn and clean up entities when Spawn throws, rather than marking failed spawns as managed.
- Native transformer persistence, bounded reconciliation, managed IO output refresh and existing electrical wiring behavior are retained.
- In-game verification required: enable the intended family in the map-specific pole-types JSON, refresh infrastructure, confirm candidate counts, test transformer visibility and wiring, restart and check saved entity adoption.
- Protected CI build success does not by itself prove in-game client visibility or power propagation.

## 4.4.15 — October stabilization: scheduler overlap accounting (staged)

- Correct scheduler running-execution accounting: release one reservation rather than resetting the counter when overlapping callbacks are permitted.
- Keep scheduler public interfaces and default behavior unchanged.
- Add targeted concurrent callback and plugin unload tests to the performance audit. Plugin testing sources remain local and are not published to this repository.

## 4.4.14 — October performance audit: ImageLibrary build and clear safety (staged)

- Fix v4.4.13 CS0165 unassigned CRC compile failure.
- Add ImageLibrary clear-generation checks to abort downloads in progress and suppress callbacks/cooldowns from cleared work.
- Preserve all public SDK interfaces; runtime tests are still required for image queue teardown and callback races.

## 4.4.13 — October performance audit phase 5: ImageLibrary lifecycle (staged)

- Read ImageLibrary readiness under the same lock protecting pending work and processing state.
- Persist CommunityEntity rollover invalidation even when an image lookup misses, preventing stale metadata from remaining on disk after a wipe or entity replacement.
- Keep image queue, storage, deduplication and callback contracts unchanged; further in-flight cancellation testing remains required.

## 4.4.12 — October performance audit phase 4: HTTP metric overhead (staged)

- Batch endpoint-metric pruning every 64 records, with a cardinality safety trigger at 320 entries, rather than scanning both metric maps after every request.
- Avoid allocating a stale-host list when host-interval pruning finds no expired entries.
- Preserve request cancellation, per-host concurrency, retry behavior and public interfaces.
- Document HTTP load tests and outstanding ImageLibrary lifecycle work.

## 4.4.11 — October performance audit phase 3: cache lifecycle (staged)

- Prevent stale cache expiration and owner cleanup from deleting concurrently replaced entries using observed-entry conditional removal.
- Add a fast second cache check after a factory finishes to avoid overwriting a concurrent winner in the common case; factories may still execute concurrently and callers requiring exactly-once initialization must synchronize externally.
- Extend audit documentation with cache race regression scenarios and outstanding HTTP/ImageLibrary profiling.
- Preserve all public SDK signatures and keep plugin source local.

## 4.4.10 — October performance audit phase 2 (staged)

- Canonicalize embedded dictionary names before resource access, preventing mixed-case cache misses and redundant resource lookup failures.
- Sanitize NaN player-query radius consistently with entity scans.
- Preserve prefab-based category matches for modded entity subclasses when runtime classification differs; avoid premature category rejection.
- Guard scheduler dispatch against cancellation between timer tick and dispatch.
- Expand audit notes; runtime profiling and plugin-by-plugin validation remain required.

## 4.4.9 — Performance audit phase 1 (staged)

- Reduced categorized world scan work: known runtime types no longer require a second classifier pass or prefab lookup for filtering; unknown types retain prefab fallback.
- Treat NaN scan radii as zero to avoid malformed scans.
- Fixed case-sensitive caller dictionaries being ignored by case-insensitive single-key resolution; normal exact-key lookups remain fast.
- Added an explicit performance audit and test plan with remaining risks and profiling requirements.
- Kept existing SDK signatures, DLL dictionary assets and plugin sources unchanged.

## 4.4.8 — October Rust Gen2 compatibility and allocation cleanup (staged)

- Verified October LivestockAnimal and CritterAnimal inherit BaseNPC2; BaseNPC2 inherits BaseCombatEntity.
- Classify Gen2 livestock and critters before their NPC ancestor; retain legacy and modded fallback behavior.
- Recognize common NPC-derived player runtime names before the BasePlayer fallback.
- Reuse dictionary name validation characters to avoid allocating arrays on every lookup.
- Preserve existing public interfaces and keep plugin source local.

## 4.4.7 — Complete DeathNotes dictionary migration (staged)

- Corrected the original wooden cave spike alias and retained expanded October wildlife/livestock defaults.
- Embedded DeathNotes combat entity classification aliases alongside entity and weapon dictionaries.
- Ensured dictionary resolution validates its named embedded catalogue even if caller overrides contain the key.
- Prepared local-only RogueRustDeathNotes 2.5.0 to use DLL dictionaries exclusively for static entity, weapon and combat-type aliases; saved translations remain user-configurable.
- Synchronized DLL, runtime, CI and release identity to 4.4.7; no plugin source committed.

## 4.4.6 — World scan and dictionary hardening (staged)

- Fixed dictionary resource discovery declaration and replaced linear canonical-name lookup with a cached case-insensitive map.
- Added native categorized world scans that filter during iteration, preventing unrelated entities from consuming the 4096-result limit.
- Preserved existing world interfaces and retained the extension-method fallback for third-party world services.
- Updated DLL identity and CI/release version checks to 4.4.6.

## 4.4.5 — DeathNotes DLL catalogues and discovery optimization (staged)

- Expanded embedded DeathNotes entity defaults to cover original aliases and October wildlife/livestock additions.
- Embedded DeathNotes weapon prefab aliases in the DLL for shared use.
- Cached immutable embedded dictionary discovery to avoid repeated assembly scans and sorting.
- Prepared local-only RogueRustDeathNotes 2.4.0 integration; plugin source is not committed.
- Synchronized DLL source identity and CI/release workflow checks to 4.4.5.

## 4.4.4 — Embedded dictionary discovery (staged)

- Added DLL dictionary enumeration for plugins to discover supported embedded default sets.
- Added safe optional dictionary lookup while retaining strict GetDefaults and all existing merge/resolve APIs.
- No plugin source changes or public release; version identity synchronized across DLL and CI.

## 4.4.3 — Dictionary SDK contract completion (staged)

- Exposed existing per-key layered resolution through the public dictionary service interface.
- Added snapshot merging with server-over-plugin-over-DLL precedence for callers needing an entire dictionary.
- Preserved the existing single-override Merge overload and all plugin source remains local.
- Aligned DLL, assembly, runtime, CI and release workflow identities to 4.4.3.

## 4.4.2 — Layered dictionary resolution (staged)

- Added allocation-free per-key resolution with explicit precedence: server overrides, plugin overrides, embedded DLL defaults.
- Retained existing dictionary APIs and kept plugin source local only.
- Aligned DLL assembly, runtime, CI and release workflow version identity.

## 4.4.1 — October compatibility SDK (staged)

- Added categorized world entity queries and centralized classification.
- Added embedded, read-only dictionary defaults with override-safe merging and SDK service registration.
- Added reference manifest verification in CI.
- DLL and build infrastructure only; no plugin source is uploaded as part of this stage.

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
