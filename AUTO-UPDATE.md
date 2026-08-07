# RogueRust ProtectedCore automatic updates

## Manifest URL

RogueRust should use the permanent public manifest pointer:

```text
ROGUERUST_UPDATE_MANIFEST_URL=https://raw.githubusercontent.com/RogueAssassin/Oxide.Ext.RogueRust/main/update-manifest.json
```

This URL is maintained automatically by the private source repository after every successful protected build and public publish. Unlike GitHub's `/releases/latest/` URL, it can point to RC/prerelease builds as well as stable releases.

Automatic updates are enabled whenever that URL exists, unless explicitly disabled with:

```text
ROGUERUST_AUTO_UPDATE=false
```

Recommended settings:

```text
ROGUERUST_AUTO_UPDATE=true
ROGUERUST_UPDATE_MANIFEST_URL=https://raw.githubusercontent.com/RogueAssassin/Oxide.Ext.RogueRust/main/update-manifest.json
ROGUERUST_UPDATE_INITIAL_DELAY_MINUTES=2
ROGUERUST_UPDATE_INTERVAL_MINUTES=60
```

These variables must be present in the environment of the process that starts `RustDedicated.exe`. Setting them after the server is already running will not affect the running process.

## Windows launch script

Set the values before starting RustDedicated:

```bat
@echo off
set "ROGUERUST_UPDATE_MANIFEST_URL=https://raw.githubusercontent.com/RogueAssassin/Oxide.Ext.RogueRust/main/update-manifest.json"
set "ROGUERUST_AUTO_UPDATE=true"
set "ROGUERUST_UPDATE_INITIAL_DELAY_MINUTES=2"
set "ROGUERUST_UPDATE_INTERVAL_MINUTES=60"

RustDedicated.exe -batchmode +server.identity "server1"
```

When RogueRust Manager starts the server, add these values to the manager's server-process environment or to the wrapper batch file that the manager launches.

## Publishing a release

The private source repository is the only release authority:

```text
RogueAssassin/Oxide.Ext.RogueRust-Source
    -> protected build
    -> validation
    -> public release assets
    -> update-manifest.json on public main
    -> RogueAssassin/Oxide.Ext.RogueRust
```

The public repository does not rebuild the DLL. It hosts the already-built protected DLL, ZIP, manifests, checksums, and release metadata.

Each public release contains:

- `Oxide.Ext.RogueRust.dll`
- `RogueRust.manifest.json`
- `update-manifest.json`
- `SHA256SUMS.txt`
- `release-info.json`
- release ZIP and ZIP checksum
- `BUILD_REPORT.txt`

## Update behaviour

RogueRust checks the permanent manifest pointer after the configured initial delay and at the configured interval. When a newer compatible version is found, the DLL is downloaded, SHA-256 verified, backed up, and staged as a pending update. The running extension is not hot-swapped; the staged update is applied during the server update/restart workflow.
