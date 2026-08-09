<p align="center">
  <img src="assets/roguerust-banner.png" alt="RogueRust - Oxide / CarbonMod Extension Framework DLL" width="100%">
</p>

# RogueRust — Oxide / CarbonMod Extension Framework DLL

**RogueRust 1.8.0 Stable** is an Oxide-first extension framework for Rust servers, with CarbonMod support through Carbon's Oxide-compatible runtime. It gives plugin developers one shared, versioned DLL for UI, commands, persistence, scheduling, workload coordination, HTTP, databases, world queries, diagnostics, pooling, security, integrations and other common server infrastructure.

Instead of every plugin creating its own timers, JSON stores, CUI callback system, HTTP queues, database connections and diagnostic tooling, RogueRust provides those facilities as lifecycle-owned services behind one SDK.

> **Design contract:** Oxide/uMod is the canonical API target. Carbon compatibility is additive. RogueRust plugins should not require Carbon-specific assemblies or APIs.

## Why RogueRust exists

RogueRust is the common foundation underneath high-performance Rust plugins. It centralises infrastructure normally duplicated across plugins and automatically cleans up owned resources when a plugin unloads or a player disconnects.

Server owners get consistent diagnostics and shared infrastructure. Plugin developers can concentrate on gameplay and UI rather than repeatedly implementing framework code.

## Included in 1.8.0

| Area | What RogueRust provides |
| --- | --- |
| **RogueUI** | Oxide CUI document builder, stable callbacks/actions, per-player view state, unchanged-render suppression, conservative safe partial replacement, atomic structural rebuilds, layouts, windows, tabs, badges, toggles, inputs, progress, modals, confirmations, pagination and toast helpers. |
| **Commands** | `[RogueCommand]`, aliases, usage/description metadata, permissions, cooldowns, validation, chat/console bridges and owner cleanup. |
| **Persistence** | Typed data/configuration helpers, cache service, serialization, local SQLite and optional MySQL/MariaDB connections. |
| **Scheduling** | Delays, repeating jobs, cron/absolute scheduling, cancellation and owner-scoped cleanup. |
| **Workload coordination** | Throttle, debounce, next-tick coalescing and unique repeating work to reduce duplicate hot-hook/timer activity. |
| **HTTP & messaging** | Async HTTP helpers, request ownership/cancellation, internal transport and Discord webhook support. |
| **World services** | Player lookup, nearby players, grid conversion, terrain, entities, monuments, topology, spawns, pathfinding and terrain analysis. |
| **Developer services** | Capability/dependency registry, plugin manifests, adapters, profiling, runtime metrics, circuit breakers, logging, pooling and service exploration. |
| **Advanced utilities** | Image cache, map rendering, loot selection, entity snapshots/serialization, binary serialization and font metadata. |
| **Security & release** | Artifact-integrity service, protected release pipeline, manifests, SHA-256 release assets and update checks. |

## Runtime support

### Oxide / uMod

Oxide is RogueRust's primary runtime contract. For a normal Oxide Rust server, stop the server and copy `Oxide.Ext.RogueRust.dll` to:

```text
RustDedicated_Data/Managed/
```

Start the server and verify the extension with:

```text
roguerust.version
roguerust.status
```

### CarbonMod

RogueRust also supports Carbon through its Oxide compatibility layer. Use Carbon's Oxide-style extension location for `Oxide.Ext.RogueRust.dll`, restart the server, and use the same RogueRust plugins and SDK surface. No Carbon-specific dependency is required by the public RogueRust SDK.

Do not copy development assemblies from `References/` to a production server.

## Administrator commands

| Command | Purpose |
| --- | --- |
| `roguerust.version` | Show the loaded RogueRust version. |
| `roguerust.status` | High-level framework and RogueUI status. |
| `roguerust.kernel` | Kernel, service, capability and module health. |
| `roguerust.health` | Logger, profiler and circuit-breaker health. |
| `roguerust.readiness` | Runtime/release readiness summary. |
| `roguerust.services` | Explore registered services. |
| `roguerust.commands` | Registered commands and generated usage. |
| `roguerust.sdk` | Plugin manifests and dependency validation. |
| `roguerust.adapters` | Integration/provider status. |
| `roguerust.world` | World/entity/terrain/monument/spawn diagnostics. |
| `roguerust.jobs` | Scheduler and cron diagnostics. |
| `roguerust.network` | HTTP/internal transport diagnostics. |
| `roguerust.database` | Database and migration diagnostics. |
| `roguerust.data` | Typed data-file diagnostics. |
| `roguerust.discord` | Discord webhook diagnostics. |
| `roguerust.performance` | Runtime, GC, thread and profiler diagnostics. |
| `roguerust.logs` | Recent RogueRust warnings/errors. |
| `roguerust.pools` | Object-pool diagnostics. |
| `roguerust.security` | Artifact-integrity status. |
| `roguerust.advanced` | Advanced service diagnostics. |
| `roguerust.update` | Trigger an update check. |

## Building a RogueRust plugin

The preferred model is to inherit from `RogueRustPlugin` and reference the installed `Oxide.Ext.RogueRust.dll` alongside normal Rust/Oxide managed references.

```csharp
using Oxide.Ext.RogueRust.Plugins;
using Oxide.Ext.RogueRust.SDK;

namespace Oxide.Plugins;

[Info("RogueHello", "YourName", "1.0.0")]
[Description("Minimal RogueRust example")]
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

RogueRust handles command registration, permission, cooldown, validation and unload cleanup.

## RogueUI quick start

RogueUI is built on Rust/Oxide CUI. Structural navigation is intentionally conservative: identical documents are skipped; safe isolated leaves may be replaced selectively; tabs, modals, buttons and structural transitions rebuild atomically for client stability.

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
        .WindowHeader("Example.Header", "Example.Main", "ROGUE DASHBOARD", "Powered by RogueRust", "Example.Main")
        .Tab("Example.Overview", "Example.Main", new RogueUiRect("0.05 0.76", "0.27 0.84"), "Overview", overview, page == "overview")
        .Tab("Example.Metrics", "Example.Main", new RogueUiRect("0.29 0.76", "0.51 0.84"), "Metrics", metrics, page == "metrics");

    ShowUiIfChanged(player, ui);
}
```

Use UI state for temporary presentation state only. Persistent gameplay data belongs in the data/database services.

## Common SDK helpers

```csharp
PluginState state = LoadData("MyPlugin/state", () => new PluginState());
SaveData("MyPlugin/state", state);

SetCache("result", expensiveResult, TimeSpan.FromMinutes(10));
if (TryGetCache("result", out MyResult cached)) { }

Delay(TimeSpan.FromSeconds(5), RunOnce, "run-once");
Repeat(TimeSpan.FromMinutes(5), Maintenance, "maintenance");

Throttle("scan", TimeSpan.FromMilliseconds(250), ScanNearbyEntities);
Debounce("save", TimeSpan.FromSeconds(1), SaveState);
CoalesceNextTick("ui-refresh", RefreshUi);
RepeatUnique("maintenance", TimeSpan.FromSeconds(30), Maintenance);

string grid = GridReference(player.transform.position);
var nearby = NearbyPlayers(player.transform.position, 100f);

using (Measure("MyPlugin", "ExpensiveOperation"))
{
    RunExpensiveOperation();
}
```

## Services exposed to plugins

The complete service surface is available through `RogueServices.Instance` or the protected `Rogue` property on `RogueRustPlugin`.

`Kernel`, `Registry`, `Lifecycle`, `Modules`, `Capabilities`, `Dependencies`, `Compatibility`, `Permissions`, `Integrations`, `Players`, `Cooldowns`, `Data`, `Events`, `Configuration`, `Cache`, `Commands`, `Ui`, `Adapters`, `World`, `Entities`, `Terrain`, `Monuments`, `Topology`, `Spawns`, `Scheduler`, `Workloads`, `Http`, `Network`, `Discord`, `Logger`, `Profiler`, `RuntimeMetrics`, `CircuitBreakers`, `Database`, `Pools`, `Serialization`, `Utilities`, `PluginSdk`, `Security`, `Developer`, `BinarySerialization`, `Pathfinding`, `TerrainAnalysis`, `MapRendering`, `Images`, `Loot`, `EntitySerialization`, and `Fonts`.

Use capability checks when a plugin can operate without an optional service:

```csharp
if (HasCapability("scheduler:cron"))
{
    // Enable cron-backed functionality.
}
```

## Compatibility hooks for normal `RustPlugin` plugins

A plugin does not have to inherit from `RogueRustPlugin`. Conventional Oxide/Carbon-compatible plugins can use `Interface.CallHook(...)`.

```csharp
object version = Interface.CallHook("RogueRust_GetVersion");
object supported = Interface.CallHook("RogueRust_HasCapability", "scheduler:cron");
```

| Hook group | Available hooks |
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

The typed SDK is preferred because it provides compile-time types and richer services. Compatibility hooks are intentionally narrower.

## Database strategy

For a single server, use RogueRust's own local SQLite store. It is isolated from Oxide/Carbon internal databases and uses a game-server-oriented WAL configuration.

Use MySQL/MariaDB when data must be shared by multiple Rust servers, a website, control panel or another external process. RogueRust owns the connection/provider abstraction so individual plugins do not need separate database plumbing.

## Lifecycle ownership

Where supported, RogueRust associates resources with the owning plugin. On unload/reload it removes or cancels owned event subscriptions, cache entries, commands, UI documents/callbacks, coordinated workloads, scheduled jobs, HTTP work and database registrations. Player UI state/callbacks are also cleaned on disconnect.

## 1.x → 1.8.0 evolution

The detailed history is in [`CHANGELOG.md`](CHANGELOG.md).

- **1.0.x** — initial RogueRust 1.x framework foundation.
- **1.1.0** — shared pooling, serialization, utilities, typed data and Discord services.
- **1.2.x** — world services, runtime metrics, internal transport, manifests/dependencies, integrity/security, pathfinding, map/image/loot/entity tooling and hardened release pipeline.
- **1.3.0** — consolidated stable framework baseline after the 1.2 release-candidate series.
- **1.4.0** — workload coordinator and performance SDK.
- **1.5.0** — RogueUI performance work, secure callbacks, SQLite improvements and corrected MySQL/MariaDB provider discovery; Oxide-first architecture formalised.
- **1.6.0** — richer RogueUI state/layout/control system and UI telemetry.
- **1.7.x** — dirty-render experiments, Oxide CUI serialization corrections, dual-runtime command compatibility, client-safe reconciliation and stable server-side UI actions.
- **1.8.0 Stable** — validated 1.7.5 core plus safe high-level windows, headers, close controls, notifications, confirmation dialogs, pagination and responsive layout helpers.

## Building the extension from source

Requirements: Windows PowerShell 5.1+, a .NET SDK capable of building `net48`, current Rust/Oxide managed references in `References/`, and the bundled ConfuserEx tooling.

```powershell
.\build\Release.cmd -Version 1.8.0
```

The pipeline validates tooling, synchronises version metadata, compiles `net48`, runs repository checks, validates templates/API surfaces, protects the DLL, generates the update manifest, packages release assets and writes SHA-256/build information.

## Repository structure

```text
.github/workflows/   CI and public publishing
assets/              GitHub/public artwork
build/               release and packaging scripts
docs/                focused technical SDK/design documentation
References/          compile-time Rust/Oxide/Unity assemblies
samples/             example RogueRust plugins
scripts/             validation/build helpers
security/            signing instructions (private keys ignored)
specs/               manifest/schema formats
src/                 RogueRust extension source
templates/           plugin templates
tools/               RogueRust CLI and build tooling
```

## Support

When reporting a problem, include the RogueRust version, runtime (Oxide or Carbon), Rust server build, relevant `roguerust.status`/diagnostic output, plugin name/version using RogueRust, and the complete exception or server-console message.

Plugin authors should feature-detect optional capabilities, keep stable UI element/action keys, use RogueRust-owned scheduling/workload helpers instead of duplicate timers where appropriate, and keep Unity/Rust object access on the normal game thread unless an API explicitly documents otherwise.

## Repositories

- Public releases and updater: `https://github.com/RogueAssassin/Oxide.Ext.RogueRust`
- Source/build repository: `https://github.com/RogueAssassin/Oxide.Ext.RogueRust-Source`

## License

See [`LICENSE`](LICENSE).
