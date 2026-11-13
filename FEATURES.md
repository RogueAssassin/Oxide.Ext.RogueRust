# RogueRust 4.4.69 features

RogueRust is a shared extension: on its own it adds console diagnostics and automatic updates, but no gameplay. The features below are services that RogueRust plugins use, so each plugin doesn't carry its own copy. For what each plugin does in game, see the [plugin catalogue](https://plugins.roguegaming.com.au).

It runs on Oxide and on Carbon, through Carbon's Oxide compatibility. It doesn't need Carbon-specific files.

## Database and saved data

- One SQLite engine shared by all plugins: Rust's own `sqlite3.dll`, with versioned migrations, transactions and batched writes.
- Prepared statements are cached per connection and reused, so repeated writes don't re-parse the same SQL (new in 4.4.69).
- Database work is queued and limited per connection. `roguerust.database` shows active and queued work, failures and timings.
- Plugin data and config files are written atomically: a crash mid-write can't leave half a file. Frequent saves can be grouped into one write, or done off the main thread.
- RogueRust plugins rewrite their config only when it changed, and keep the previous file as `<plugin>.json.v<old version>.bak` when the plugin version changes (new in 4.4.69).

## Images

- Image downloads for plugin UIs, at most three at a time, stored in Rust's server image store.
- Each image is stored once, even when several plugins or URLs use the same picture (SHA-256 content check).
- PNG files are validated and stored as they are, not re-encoded. Other formats are converted.
- Icons built into the DLL need no download.
- A URL that keeps failing is paused for five minutes rather than retried on every request.
- Temporary failures (connection error, timeout, rate limit, server error) are retried after 2, 8 and 30 seconds (new in 4.4.69).
- When the server entity that owns the image store changes, stored images are moved over, so the image IDs plugins hold stay valid (new in 4.4.69).
- Images are tracked per plugin; `roguerust.performance` shows how many each plugin has registered and stored, and roughly how much space they use.

## Workshop and skins

- Workshop item and collection details are looked up by ID without a Steam API key. Results are kept in `RogueRust/Assets/RogueAssets.db` and refreshed after seven days.
- Lookups are batched, and a request already in progress is shared, not repeated.
- A shared skin catalogue for all items, searchable by name, Workshop ID or content ID, plus building-grade skins.
- On restart, an unchanged skin catalogue isn't written back to disk (new in 4.4.69).
- Applying a skin to a held item updates what players see straight away.
- Plugins decide who may use which skin. RogueRust only provides the catalogue and applies the skin.

## UI

- Shared UI building blocks for plugin menus: layouts, tables, tiles, search and paging.
- Each menu button runs through a random token that belongs to one player and expires, so other players can't trigger it by typing the console command.
- Only the changed parts of a menu are re-sent.

## Server and world helpers

- Grid, monument, road, river, rail and terrain lookups, and spatial entity queries.
- Teleport destinations, with a safe-position check and a previous-location record for "back".
- Vehicle catalogue (cars, boats, helicopters, trains, animals and more) for admin menus.
- Power-pole discovery and state for GridPower.

## Running safely

- One shared scheduler and workload system, so plugins don't each run their own timers.
- Limited, retrying HTTP requests, with circuit breakers that pause a failing service.
- Load is tracked per plugin, and a plugin using too much is flagged in `roguerust.performance`.
- On shutdown, queued work finishes before the extension unloads.
- Discord webhook delivery with rate-limit handling.

## Updates and diagnostics

- Signed automatic updates from this repository's releases; see [AUTO-UPDATE.md](AUTO-UPDATE.md).
- Console diagnostics: `roguerust.version`, `status`, `health`, `performance`, `database`, `update`; `roguerust.commands` lists the rest.
