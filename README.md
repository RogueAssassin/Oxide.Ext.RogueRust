# RogueRust 4.4.69

**Shared extension for Rust servers running Oxide or Carbon**

RogueRust is the extension DLL that the [RogueRust plugins](https://github.com/RogueAssassin/RogueRust-Plugins) are built on. It gives them one shared set of services: UI, images, SQLite storage, Workshop and skin data, HTTP, scheduling, permissions and diagnostics. Each plugin then doesn't run its own copy of the same timers, downloads and database work.

Plugin catalogue and documentation: **[plugins.roguegaming.com.au](https://plugins.roguegaming.com.au)**

## Install

1. Download `Oxide.Ext.RogueRust.dll` from the [latest release](https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/latest).
2. Put it in the extensions folder:
   - **Oxide:** `RustDedicated_Data/Managed/`
   - **Carbon:** `carbon/extensions/`
3. Restart the server. Extensions only load at startup.
4. Install the plugins you want from the [plugin repository](https://github.com/RogueAssassin/RogueRust-Plugins). The current plugins need **RogueRust 4.4.69 or newer**.

To check the download, each release includes `SHA256SUMS.txt` and a signed `RogueRust.manifest.json`.

## What changed from 4.4.68 to 4.4.69

Measured on the SQLite engine Rust ships with (`sqlite3.dll` from the dedicated server). Each figure is the median of 7 runs of the skin-cache rewrite RogueRust does after a catalogue rebuild:

| Skins in catalogue | 4.4.68 | 4.4.69 | Faster by |
|---:|---:|---:|---:|
| 2,000 | 31.1 ms | 11.9 ms | 2.6× |
| 6,698 (a full current catalogue) | 97.8 ms | 39.6 ms | 2.5× |
| 20,000 | 292.4 ms | 134.9 ms | 2.2× |

- **Database:** each SQL statement is prepared once per connection and reused, instead of being prepared again for every row.
- **Skins:** when the skin catalogue hasn't changed since the last boot, it is no longer written back to disk and the database on restart.
- **Plugin configs:** RogueRust plugins now rewrite their config file only when something in it changed. Before 4.4.69, 16 config files were rewritten on every restart. When a plugin updates to a new version, its old config is kept beside the new one as `<plugin>.json.v<old version>.bak`.
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
| `roguerust.performance` | Scheduler, memory, UI and image-library load, broken down by plugin |
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
