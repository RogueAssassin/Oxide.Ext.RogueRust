<p align="center">
  <img src="assets/roguerust-banner.png" alt="RogueRust - Oxide / CarbonMod Extension Framework DLL" width="100%">
</p>

# RogueRust — Oxide / CarbonMod Extension Framework DLL

**RogueRust 2.1.0** is an Oxide-first extension framework for Rust servers, with CarbonMod support through its Oxide-compatible runtime surface. It provides one shared DLL for plugin UI, commands, scheduling, workload coordination, databases, HTTP, image caching, world queries, diagnostics, lifecycle ownership, pooling, security and common developer infrastructure.

RogueRust is not intended to replace Oxide or Carbon. It sits above the normal Rust/Oxide plugin environment and gives RogueRust-aware plugins a consistent SDK so they do not each need to implement their own CUI framework, command router, timers, HTTP queues, database plumbing and cleanup logic.

> **Runtime contract:** Oxide/uMod is the canonical API target. Carbon support is compatibility-first and should not require RogueRust plugins to compile against Carbon-specific assemblies.

## What changed from 1.8.0 to 2.1.0

The 1.8.0 release established the client-safe RogueUI and stable callback baseline. The 1.9–2.1 line expands that stable foundation rather than replacing it.

- **1.8.0 Stable** — validated RogueUI reconciliation, stable server-side UI actions, windows/headers, notifications, confirmations, pagination and responsive layouts.
- **1.9.0** — replaced extension-side host command registration with native Rust hook command dispatch. RogueRust no longer depends on `AddChatCommand`/`AddConsoleCommand` overload compatibility.
- **1.9.1** — introduced the native FileStorage-backed ImageLibrary, native `ItemIcon(itemId, skinId)` support and direct host configuration layout/migration.
- **2.0.x** — unified the 2.x release baseline and made Unity `ImageConversion` runtime-resolved so RogueRust stays net48-compatible with current Rust server assemblies.
- **2.1.0** — performance hardening: lazy ImageLibrary validation, debounced/atomic image metadata writes, source/decode limits, cached ImageConversion reflection, globally bounded HTTP execution, more efficient database batching, public documentation cleanup and a dedicated samples release asset.

See [CHANGELOG.md](CHANGELOG.md) for the full 1.x → 2.1.0 history and [FEATURES.md](FEATURES.md) for the complete feature catalogue.

## Why server owners use RogueRust

RogueRust centralises infrastructure normally duplicated by individual plugins. When compatible plugins use the framework, the server gains one lifecycle owner for common resources, one diagnostic surface and fewer independently implemented timers, queues, databases and UI callback systems.

The performance philosophy is conservative: skip work when nothing changed, batch related work, bound network concurrency, keep Unity/Rust access on the game thread, and prefer predictable client-safe CUI operations over risky micro-optimisations.

## Installation

### Oxide / uMod

Stop the server and install the public `Oxide.Ext.RogueRust.dll` in the normal Oxide extension/managed location used by your server build. Start the server and verify:

```text
roguerust.version
roguerust.status
roguerust.readiness
```

### CarbonMod

Use Carbon's Oxide-compatible extension loading location for `Oxide.Ext.RogueRust.dll`, then restart the server. RogueRust plugins use the same public SDK on both runtimes.

Do **not** copy development-only assemblies from the source `References/` folder into production plugin folders.

## Public release downloads

Each public 2.1.x GitHub release is intended to provide:

- `Oxide.Ext.RogueRust.dll` — protected framework DLL for server installation.
- `Oxide.Ext.RogueRust-v2.1.0-release.zip` — DLL, manifests, README, changelog, features and license.
- `Oxide.Ext.RogueRust-v2.1.0-samples.zip` — example RogueRust plugins and project templates.
- SHA-256 checksum files and release/build metadata.

**Samples:**
`https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/download/v2.1.0/Oxide.Ext.RogueRust-v2.1.0-samples.zip`

Public releases:
`https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases`

## Main framework areas

| Area | What it provides |
| --- | --- |
| **RogueUI** | Rust/Oxide CUI documents, stable callbacks/actions, per-player UI state, unchanged-render suppression, safe reconciliation, windows, tabs, controls, modals, pagination, notifications and layouts. |
| **ImageLibrary** | Native remote-image queue/cache backed by Rust FileStorage, CRC reuse, stale-entry recovery, item/skin icon helpers and 2.1.0 performance/memory limits. |
| **Commands** | `[RogueCommand]`, aliases, permissions, cooldowns, typed arguments, validation and native Rust-hook dispatch. |
| **Data/config** | Typed plugin data/configuration helpers plus shared cache and migration-aware configuration handling. |
| **Databases** | Local SQLite plus optional MySQL/MariaDB, migrations, parameterized queries and transaction-backed batches. |
| **Scheduler/workloads** | Delay/repeat/cron jobs, throttle, debounce, coalescing and unique repeated workloads. |
| **HTTP/networking** | Async HTTP, retry/cancellation, per-host pacing, Discord support and global bounded execution. |
| **World services** | Player, entity, terrain, monument, topology, spawn, map/grid, pathfinding and terrain-analysis helpers. |
| **Diagnostics** | Service/kernel health, runtime metrics, profiler, logs, pools, circuit breakers, DB/network/UI diagnostics and readiness checks. |
| **Developer platform** | Capability registry, dependency manifests, adapters, plugin SDK, lifecycle cleanup, serialization and advanced utility services. |

## Native ImageLibrary — SteamID question

**A player SteamID is not required. A Steam Web API key is not required by the core ImageLibrary either.**

RogueRust stores normalized images in Rust `FileStorage` against the server `CommunityEntity`. The optional `imageId` argument is only a cache variant identifier—commonly useful for a skin/workshop ID—and is not treated as player identity.

Typical usage:

```csharp
string? crc = ImageLibrary.GetOrQueue(
    "event.logo",
    "https://cdn.example.com/event-logo.png",
    imageId: 0,
    callback: _ => RedrawUi(player));
```

Then use the returned FileStorage CRC in RogueUI:

```csharp
if (crc != null)
    ui.Image("MyPlugin.Logo", "MyPlugin.Main", new RogueUiRect("0.05 0.60", "0.25 0.90"), crc);
```

For normal Rust item/skin previews, prefer native item rendering where practical:

```csharp
ui.ItemIcon("MyPlugin.Item", "MyPlugin.Main", rect, itemId, skinId);
```

### ImageLibrary performance behavior in 2.1.0

- FileStorage PNG validation occurs lazily once per cached CRC/runtime community entity; normal later reads are dictionary-only.
- Stale CRCs are removed from metadata and can be re-downloaded from their registered URL.
- Remote source payloads over **8 MiB** are rejected before decode.
- Decoded images over **4096×4096** are rejected.
- Normalized PNGs over **3 MiB** are rejected.
- Unity ImageConversion reflection methods are cached after first successful discovery.
- Metadata changes are coalesced into a short delayed save instead of rewriting `images.json` for every completed image.
- Metadata is serialized from a true locked snapshot and written through a temporary file/replace path.
- Downloads remain deliberately sequential in the image queue to avoid Texture2D/decode memory spikes on the game server.

## Administrator commands

| Command | Purpose |
| --- | --- |
| `roguerust.version` | Loaded framework version. |
| `roguerust.status` | High-level framework and RogueUI status. |
| `roguerust.readiness` | Release/runtime readiness summary. |
| `roguerust.kernel` | Kernel, modules, services and capability health. |
| `roguerust.health` | Logger/profiler/circuit-breaker health. |
| `roguerust.performance` | Runtime, GC, thread and profiler diagnostics. |
| `roguerust.services` | Registered service exploration. |
| `roguerust.commands` | Registered RogueRust command metadata. |
| `roguerust.sdk` | Plugin manifests/dependency information. |
| `roguerust.adapters` | Adapter/provider status. |
| `roguerust.world` | World/entity/terrain/monument/spawn diagnostics. |
| `roguerust.jobs` | Scheduler diagnostics. |
| `roguerust.network` | HTTP/internal transport diagnostics. |
| `roguerust.database` | Database/migration diagnostics. |
| `roguerust.data` | Typed data/config diagnostics. |
| `roguerust.discord` | Discord integration diagnostics. |
| `roguerust.logs` | Recent RogueRust warnings/errors. |
| `roguerust.pools` | Object-pool diagnostics. |
| `roguerust.security` | Artifact-integrity status. |
| `roguerust.advanced` | Advanced services status. |
| `roguerust.update` | Trigger an update check. |
| `roguerust.exec <command> [args...]` | Execute a RogueRust command from local console/RCON. |

Internal UI commands (`roguerust.ui.callback` and `roguerust.ui.close`) are framework plumbing and are not intended as normal administrator commands.

## Building a RogueRust plugin

The preferred model is to inherit from `RogueRustPlugin` and reference the installed `Oxide.Ext.RogueRust.dll` alongside your normal Rust/Oxide managed references.

```csharp
using Oxide.Ext.RogueRust.Plugins;
using Oxide.Ext.RogueRust.SDK;

namespace Oxide.Plugins;

[Info("RogueHello", "YourName", "2.1.0")]
[Description("Minimal RogueRust command example")]
public sealed class RogueHello : RogueRustPlugin
{
    [RogueCommand(
        "rhello",
        Aliases = new[] { "rh" },
        Description = "Displays a RogueRust greeting.",
        Usage = "/rhello",
        Permission = "roguehello.use",
        CooldownSeconds = 2)]
    private RogueCommandResult Hello(RogueCommandContext context)
    {
        string who = context.Player?.Name ?? "server";
        return RogueCommandResult.Ok($"Hello {who} from RogueRust.");
    }
}
```

RogueRust owns command metadata and applies permission/cooldown/argument handling through its native command pipeline.

## Native command lifecycle

RogueRust no longer depends on extension-side `AddChatCommand`, `AddConsoleCommand`, `RemoveChatCommand` or `RemoveConsoleCommand` overloads.

- player chat commands dispatch through Rust command hooks before normal fallback;
- RCON commands dispatch through RogueRust's RCON hook path;
- local server console/RCON can use `roguerust.exec`;
- unknown commands continue into the normal Rust/Oxide/Carbon pipeline.

This avoids the runtime CLR overload mismatch that affected earlier dual-runtime command registration.

## RogueUI example

```csharp
private void OpenDashboard(BasePlayer player)
{
    string overview = UiActionCallback(player, "page.overview", () =>
    {
        SetUiState(player, "page", "overview");
        OpenDashboard(player);
    });

    string metrics = UiActionCallback(player, "page.metrics", () =>
    {
        SetUiState(player, "page", "metrics");
        OpenDashboard(player);
    });

    string page = GetUiState(player, "page", "overview") ?? "overview";

    RogueUiDocument ui = CreateUi("Example.Main")
        .Panel("Example.Main", "Overlay", RogueUiRect.Centered(0.58f, 0.56f), cursorEnabled: true)
        .WindowHeader("Example.Header", "Example.Main", "ROGUE DASHBOARD", "Example.Main", "Powered by RogueRust")
        .Tab("Example.Overview", "Example.Main", new RogueUiRect("0.05 0.76", "0.27 0.84"), "Overview", overview, page == "overview")
        .Tab("Example.Metrics", "Example.Main", new RogueUiRect("0.29 0.76", "0.51 0.84"), "Metrics", metrics, page == "metrics");

    ShowUiIfChanged(player, ui);
}
```

RogueUI's stable policy is intentionally conservative:

- identical document → **send nothing**;
- one safe isolated leaf change → **selective replacement may be used**;
- navigation, buttons, inputs, modals, hierarchy/structural changes or multiple coupled changes → **atomic document rebuild**.

That policy was chosen after live-client testing exposed the risks of aggressively using Rust CUI `update:true` on arbitrary objects.

## Scheduler and workload helpers

```csharp
Delay(TimeSpan.FromSeconds(5), RunOnce, "run-once");
Repeat(TimeSpan.FromMinutes(5), Maintenance, "maintenance");

Throttle("scan", TimeSpan.FromMilliseconds(250), ScanNearbyEntities);
Debounce("save", TimeSpan.FromSeconds(1), SaveState);
CoalesceNextTick("ui-refresh", RefreshUi);
RepeatUnique("maintenance", TimeSpan.FromSeconds(30), Maintenance);
```

These are preferred over creating multiple independent timers for the same logical work.

## Database usage

For a single local Rust server, Rogue-owned SQLite is the normal default. Use MySQL/MariaDB when data must be shared between multiple servers, a website/control panel or other external services.

For groups of related writes, prefer `ExecuteBatchAsync`:

```csharp
await Database.ExecuteBatchAsync("default", new[]
{
    new RogueDatabaseStatement("UPDATE player_stats SET kills = kills + 1 WHERE steam_id = @id",
        new[] { new RogueDatabaseParameter("id", steamId) }),
    new RogueDatabaseStatement("INSERT INTO audit_log (steam_id, action) VALUES (@id, @action)",
        new[]
        {
            new RogueDatabaseParameter("id", steamId),
            new RogueDatabaseParameter("action", "kill")
        })
}, transaction: true);
```

In 2.1.0, ADO/MySQL batches execute through **one worker hop per batch**, rather than one `Task.Run` per statement. SQLite continues using the framework-owned SQLite execution path.

## HTTP performance behavior

RogueRust HTTP is asynchronous from plugin callers, but the underlying .NET request API is blocking. In 2.1.0 RogueRust places a global `SemaphoreSlim` gate in front of execution so at most **8 active blocking HTTP attempts** consume worker threads at once.

Per-host pacing, cancellation, retry policy and owner cleanup still apply. This protects the CLR ThreadPool if many plugins all make requests at the same time.

## Common SDK helpers

```csharp
PluginState state = LoadData("MyPlugin/state", () => new PluginState());
SaveData("MyPlugin/state", state);

SetCache("result", expensiveResult, TimeSpan.FromMinutes(10));
if (TryGetCache("result", out MyResult cached)) { }

string grid = GridReference(player.transform.position);
var nearby = NearbyPlayers(player.transform.position, 100f);

using (Measure("MyPlugin", "ExpensiveOperation"))
{
    RunExpensiveOperation();
}
```

## Services exposed to plugins

The complete service surface is available through `RogueServices.Instance` or the protected `Rogue` property on `RogueRustPlugin`.

Major services include `Kernel`, `Registry`, `Lifecycle`, `Modules`, `Capabilities`, `Dependencies`, `Compatibility`, `Permissions`, `Integrations`, `Players`, `Cooldowns`, `Data`, `Events`, `Configuration`, `Cache`, `Commands`, `Ui`, `Adapters`, `World`, `Entities`, `Terrain`, `Monuments`, `Topology`, `Spawns`, `Scheduler`, `Workloads`, `Http`, `Network`, `Discord`, `Logger`, `Profiler`, `RuntimeMetrics`, `CircuitBreakers`, `Database`, `Pools`, `Serialization`, `Utilities`, `PluginSdk`, `Security`, `Developer`, `ImageLibrary`, `BinarySerialization`, `Pathfinding`, `TerrainAnalysis`, `MapRendering`, `Images`, `Loot`, `EntitySerialization` and `Fonts`.

Feature-detect optional functionality where appropriate:

```csharp
if (HasCapability("scheduler:cron"))
{
    // Enable cron-specific behavior.
}
```

## Compatibility hooks for normal RustPlugin plugins

A plugin does not have to inherit from `RogueRustPlugin`. Normal Oxide/Carbon-compatible plugins can call the narrower compatibility surface through `Interface.CallHook(...)`.

```csharp
object version = Interface.CallHook("RogueRust_GetVersion");
object supported = Interface.CallHook("RogueRust_HasCapability", "scheduler:cron");
```

Public hook groups include:

| Group | Hooks |
| --- | --- |
| Framework | `RogueRust_GetVersion`, `RogueRust_IsPluginLoaded`, `RogueRust_Call`, `RogueRust_IsCompatible` |
| Capabilities/dependencies | `RogueRust_HasCapability`, `RogueRust_GetCapabilityProvider`, `RogueRust_AreDependenciesSatisfied` |
| Players/cooldowns | `RogueRust_FindPlayer`, `RogueRust_GetCooldownRemaining`, `RogueRust_TryStartCooldown`, `RogueRust_ClearCooldown` |
| Data/config/cache | `RogueRust_DataExists`, `RogueRust_DeleteData`, `RogueRust_ConfigExists`, `RogueRust_DeleteConfig`, `RogueRust_CacheContains`, `RogueRust_RemoveCache`, `RogueRust_ClearExpiredCache` |
| Commands/UI | `RogueRust_ExecuteCommand`, `RogueRust_GetCommandCount`, `RogueRust_DestroyUi`, `RogueRust_IsUiOpen`, `RogueRust_GetOpenUiCount` |
| Adapters/economy | `RogueRust_HasAdapter`, `RogueRust_GetAdapterProvider`, `RogueRust_GetEconomyBalance`, `RogueRust_DepositEconomy`, `RogueRust_WithdrawEconomy` |
| World | `RogueRust_PositionToGrid`, `RogueRust_TryGridToWorld`, `RogueRust_GetNearbyPlayerCount`, `RogueRust_GetTerrainHeight` |
| Scheduler/HTTP | `RogueRust_CancelScheduledJob`, `RogueRust_CancelScheduledJobsByOwner`, `RogueRust_GetScheduledJobCount`, `RogueRust_GetHttpRequestCount`, `RogueRust_GetActiveHttpRequestCount`, `RogueRust_CancelHttpRequestsByOwner` |
| Diagnostics/security | `RogueRust_GetProfiledOperationCount`, `RogueRust_GetSlowOperationCount`, `RogueRust_IsCircuitAvailable`, `RogueRust_ResetCircuit`, `RogueRust_HasPermission` |
| Database | `RogueRust_GetDatabaseConnectionCount`, `RogueRust_IsDatabaseRegistered` |

The typed SDK is preferred for RogueRust-aware plugins; compatibility hooks intentionally expose a smaller surface.

## Lifecycle ownership

Where supported, RogueRust associates resources with an owning plugin. On unload/reload it removes or cancels owned event subscriptions, cache entries, commands, UI state/callbacks, coordinated workloads, scheduled jobs, HTTP requests and database registrations. Player UI state/callbacks are also cleaned on disconnect.

## Performance guidance for plugin authors

- Do not run expensive scans in every hot Rust hook; throttle/coalesce them.
- Do not rewrite unchanged UI every tick; use the RogueUI update helpers and stable element keys.
- Use stable action callbacks for finite UI state transitions.
- Batch DB writes when they belong to the same logical update.
- Keep BasePlayer/BaseEntity/FileStorage/Texture2D/Unity interaction on the game thread unless a specific RogueRust API documents otherwise.
- Reuse ImageLibrary CRCs; do not download images during every render.
- Prefer native `ItemIcon` for item/skin previews where possible.
- Use the shared HTTP service instead of giving every plugin an independent blocking network worker model.

## Samples package

The public samples package contains:

- `RogueRustExample` — broad SDK tour.
- `RogueRustMariaDbExample` — optional MySQL/MariaDB registration and migrations.
- `RogueDatabaseBatchExample` — 2.1 transaction/batch usage.
- `RogueImageLibraryExample` — native FileStorage image caching; no SteamID required.
- `RogueUiShowcase` — compact UI composition/callback example.
- `RogueUiDashboard` — state, tabs, grid, toggles, input and progress.
- `RogueUiDirtyDashboard` — UI reconciliation/navigation regression test.
- basic, command, database and UI plugin templates.

Download:
`https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/download/v2.1.0/Oxide.Ext.RogueRust-v2.1.0-samples.zip`

The ZIP contains `SAMPLES_GUIDE.txt` with installation notes and a description of each example.

## Building the extension from source

Requirements:

- Windows PowerShell 5.1+
- .NET SDK capable of building `net48`
- current Rust/Oxide managed references in `References/`
- release protection tooling configured for protected public builds

Refresh references from your own server installation before building:

```powershell
.\scripts\Update-References.ps1 -ManagedPath "C:\path\to\RustDedicated_Data\Managed" -Clean
```

Build 2.1.0:

```powershell
.\build\Release.cmd -Version 2.1.0
```

The release pipeline now creates both the normal release ZIP and the separate samples ZIP, generates SHA-256 checksums, and the public publish workflow uploads both assets.

## Documentation layout

The public/source root is deliberately kept simple:

- `README.md` — installation, usage, SDK introduction and support information.
- `CHANGELOG.md` — full release history.
- `FEATURES.md` — current capability catalogue.
- `SAMPLES_GUIDE.txt` — plain-text guide included with sample packaging.

Historical migration/fix markdown files have been consolidated into those three public Markdown documents.

## Support

When reporting a problem, provide:

- RogueRust version;
- runtime: Oxide or Carbon;
- Rust server build;
- plugin name/version using RogueRust;
- output from the relevant `roguerust.*` diagnostic command;
- the complete exception/server-console message;
- reproduction steps, especially for UI or lifecycle issues.

## Repositories

Public releases/updater:
`https://github.com/RogueAssassin/Oxide.Ext.RogueRust`

Source/build repository:
`https://github.com/RogueAssassin/Oxide.Ext.RogueRust-Source`

## License

See `LICENSE`.
