# RogueRust 4.4.76

**Shared extension for Rust servers running Oxide or Carbon** · created by RogueAssassin

RogueRust is the extension DLL that the [RogueRust plugins](https://github.com/RogueAssassin/RogueRust-Plugins) are built on. It gives them one shared set of services: UI, images, SQLite storage, Workshop and skin data, HTTP, scheduling, permissions and diagnostics. Each plugin then doesn't run its own copy of the same timers, downloads and database work.

Plugin catalogue and documentation: **[plugins.roguegaming.com.au](https://plugins.roguegaming.com.au)**

## Install

1. Download `Oxide.Ext.RogueRust.dll` from the [latest release](https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/latest).
2. Put it in the extensions folder:
   - **Oxide:** `RustDedicated_Data/Managed/`
   - **Carbon:** `carbon/extensions/`
3. Restart the server. Extensions only load at startup.
4. Install the plugins you want from the [plugin repository](https://github.com/RogueAssassin/RogueRust-Plugins). The current plugins need **RogueRust 4.4.75 or newer**; 4.4.76 is recommended.

To check the download, each release includes `SHA256SUMS.txt` and a signed `RogueRust.manifest.json`.

## What's new in 4.4.76

- **Real process numbers in `roguerust.performance` and `roguerust.soak`.** Working set, private memory and thread count used to show 0, because Rust's Mono runtime doesn't report them through .NET. RogueRust now reads them from the operating system (Windows memory counters and thread list, or `/proc/self/status` on Linux). A value that still can't be read shows as `unavailable`.
- **Easier-to-read reports.** Uptime and durations show whole seconds (`0:24:02`, or `2d 03:04:05`). The soak report shows one GC count, as the performance report already does. When the scheduler has slow runs, the report names the slowest scheduled job and the plugin that owns it.
- **Thread warning on growth only.** Rust itself runs well over 100 threads, so the warning now appears only when the thread count rises by 50 or more over at least 30 minutes.
- **Paired plugins:** CustomPlayerMessages 3.1.8 (the first join after a load no longer takes about 20 ms) and AdminMenu 2.5.15 (vanish messages say "vanish" instead of an old module name).

## 4.4.75

- **Default permissions by level.** The first time a plugin loads, its player permissions are granted to Oxide's `default` group (every player) and its admin permissions to the `admin` group. Players get the everyday features: Furnace Splitter, normal removal, basic skinning, Building Manager skins and wallpaper, crate reports and death notices. Administrative permissions go to admins only.
- **Granted once, never again.** Each permission is recorded in `oxide/data/RogueRust/permission_defaults.json` after it is handled. Revoke one and it stays revoked, and a permission you had already granted to a group is left as you set it. Opt-outs and rule bypasses, such as free upgrades, the removal override and hiding from join messages, are never granted automatically.
- **Turn it off** by setting `Auto Grant Default Permissions` to `false` in that file. Each plugin load logs the permissions it granted in one line.
- Plugin authors declare the level on the field: `[RoguePermission(Default = RoguePermissionLevel.Player)]`.

## 4.4.74

- **Fewer disk flushes.** A data save is skipped when the file already holds exactly that content. Plugins save their data every time they unload, and most of those saves change nothing, but each one used to write the file and flush it to disk on the main thread (FurnaceSplitter's unload took about 300 ms). A file you edit or replace yourself is still overwritten as before.
- **Smaller UI pages.** New UI elements leave out zero offsets, as Oxide's own CUI helper does. That saves about 36 bytes per element, around 13% of a full AdminMenu item page. Opening a panel no longer serializes every element a second time.
- **Cheaper spawn and kill hooks.** The event type of each entity class is worked out once and cached, instead of six type tests for every entity spawned or killed on the server.
- **GridPower streetlight cache follows the map.** The cache is tied to the map seed (or the custom map), not only its size, so a wipe to a new seed at the same size rebuilds it. The first start on 4.4.74 rebuilds the cache once.
- **Paired plugins:** AdminMenu 2.5.13 rebuilds vanish on Rust's own admin invisibility. It adds a dashboard toggle and persistence through reconnects and restarts, and blocks turrets from targeting vanished admins. GridPower 2.0.9 and RemovalTool 2.0.7 are also updated.

## 4.4.73 and 4.4.72

- **`roguerust.performance` and `roguerust.status` show their full report**, and also write it to `RustDedicated.log`. Rust's console only kept the last line a command replied with. Process metrics are read separately, and the memory warning appears only on real growth (512 MiB or more over at least 30 minutes).
- **Lighter on the main thread:** hot paths skip constant and control-flow obfuscation, GridPower stops rescanning maps with no access points, and world searches and the skin catalogue do less work per entity.
- **Paired plugins:** Skins 2.3.9 (browser box security fix and retargeting), DeathNotes 2.5.9, FurnaceSplitter 2.1.9, FuelStackOptimizer 2.1.7, ServerRestarter 2.1.7.

## 4.4.71

- **Plugin configs are updated in place.** When a plugin updates, the settings already in your config file keep their values and only the settings new in that release are added. No `<plugin>.json.v<old version>.bak` copy is made any more. The server log lists the settings that were added, and a warning names any existing setting the plugin had to correct, such as an out-of-range value.
- **Lists in your config stay as you wrote them.** The paired plugin releases read lists from the file instead of merging them with the defaults, so entries are no longer duplicated, re-sorted or put back after you remove them.
- Existing `.bak` files from 4.4.69 and 4.4.70 are left where they are; delete them from `oxide/config` once you no longer need them.

4.4.70 made GridPower reloads faster: finding the power access points on a large map dropped from about 225 ms to under 4 ms.

## What changed from 4.4.68 to 4.4.69

Measured on the SQLite engine Rust ships with (`sqlite3.dll` from the dedicated server). Each figure is the median of 7 runs of the skin-cache rewrite RogueRust does after a catalogue rebuild:

| Skins in catalogue | 4.4.68 | 4.4.69 | Faster by |
|---:|---:|---:|---:|
| 2,000 | 31.1 ms | 11.9 ms | 2.6× |
| 6,698 (a full current catalogue) | 97.8 ms | 39.6 ms | 2.5× |
| 20,000 | 292.4 ms | 134.9 ms | 2.2× |

- **Database:** each SQL statement is prepared once per connection and reused, instead of being prepared again for every row.
- **Skins:** when the skin catalogue hasn't changed since the last boot, it is no longer written back to disk and the database on restart.
- **Plugin configs:** RogueRust plugins now rewrite their config file only when something in it changed. Before 4.4.69, 16 config files were rewritten on every restart. (4.4.69 and 4.4.70 also kept the old config as `<plugin>.json.v<old version>.bak` on a version change; 4.4.71 stops doing that.)
- **Workshop:** item lookups for Workshop skins no longer run on a background thread, where Rust's item manager isn't safe to use.
- **Images:** a download that fails for a temporary reason (connection error, timeout, rate limit, server error) is retried after 2, 8 and 30 seconds. When the server entity that owns the image store changes, stored images are moved to the new owner, so the image IDs plugins hold stay valid. A plugin waiting on an image always gets an answer, even when the image is in its failure cooldown or the cache is cleared.

The full history is in [CHANGELOG.md](CHANGELOG.md).

## Automatic updates

RogueRust checks this repository's latest release at startup and then every 60 minutes. When there's a newer version, it:

1. checks the release manifest's signature;
2. downloads the DLL and checks its SHA-256;
3. keeps the current DLL as `Oxide.Ext.RogueRust.dll.backup`;
4. installs the new one.

The new version loads on the **next restart**; it is never swapped in while the server is running.

The settings are in `RogueRust.Update.json`, in the Oxide/Carbon config folder. Run `roguerust.update` in the server console to check now. See [AUTO-UPDATE.md](AUTO-UPDATE.md) for details.

## Console commands

All commands are run from the server console (RCON or the server window).

| Command | Shows |
|---|---|
| `roguerust.version` | Installed version |
| `roguerust.status` | Overall status of the extension and its services |
| `roguerust.health` | Operation, slow-operation and failure counts, plus the state of each network circuit |
| `roguerust.performance` | Scheduler, memory, threads, UI and image-library load, broken down by plugin |
| `roguerust.soak` | Memory, process and queue peaks across the sampled history |
| `roguerust.database` | Database connections, queued work, failures and timings |
| `roguerust.update` | Checks for an update now (server administrators only) |
| `roguerust.commands` | Every command in the installed build |

Use `roguerust.commands` to see the full list; it is always correct for the build you have installed.

## Features

See [FEATURES.md](FEATURES.md) for what RogueRust provides to plugins.

## Support

Report problems with the DLL in [Issues](https://github.com/RogueAssassin/Oxide.Ext.RogueRust/issues). Include:

- the output of `roguerust.version`;
- whether you run Oxide or Carbon;
- the relevant lines from the server log.

For problems with a specific plugin, use the [plugin repository](https://github.com/RogueAssassin/RogueRust-Plugins/issues).

## License

See [LICENSE](LICENSE). All rights reserved by the copyright holder.
