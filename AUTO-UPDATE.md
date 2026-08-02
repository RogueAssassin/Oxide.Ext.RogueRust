# RogueRust ProtectedCore automatic updates

## Manifest URL

ProtectedCore reads the updater URL from the process environment variable:

```text
ROGUERUST_UPDATE_MANIFEST_URL=https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/latest/download/update-manifest.json
```

Automatic updates are enabled whenever that URL exists, unless explicitly disabled with:

```text
ROGUERUST_AUTO_UPDATE=false
```

Recommended settings:

```text
ROGUERUST_AUTO_UPDATE=true
ROGUERUST_UPDATE_INITIAL_DELAY_MINUTES=2
ROGUERUST_UPDATE_INTERVAL_MINUTES=60
```

These variables must be present in the environment of the process that starts `RustDedicated.exe`. Setting them after the server is already running will not affect the running process.

## Windows launch script

Add the variables before the line that starts RustDedicated:

```bat
@echo off
set "ROGUERUST_UPDATE_MANIFEST_URL=https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/latest/download/update-manifest.json"
set "ROGUERUST_AUTO_UPDATE=true"
set "ROGUERUST_UPDATE_INITIAL_DELAY_MINUTES=2"
set "ROGUERUST_UPDATE_INTERVAL_MINUTES=60"

RustDedicated.exe -batchmode +server.identity "server1"
```

When RogueRust Manager starts the server, add these values to the manager's server-process environment or to the wrapper batch file that the manager launches.

## Publishing a release

1. Replace the root `Oxide.Ext.RogueRust.dll` with the newly built and protected DLL.
2. Commit and push the DLL to `main`.
3. Confirm the DLL assembly version matches the intended release version.
4. Create and push a tag:

```powershell
git checkout main
git pull
git tag -a v1.9.0 -m "RogueRust 1.9.0"
git push origin v1.9.0
```

The `Publish RogueRust Release` workflow will:

- validate the DLL and tag versions match;
- calculate the DLL SHA-256;
- generate `update-manifest.json`;
- generate `SHA256SUMS.txt` and `release-info.json`;
- create or update the GitHub Release;
- attach the direct DLL and generated updater assets;
- verify the published manifest and hash.

The workflow can also be started manually from **Actions → Publish RogueRust Release → Run workflow**. Enter the version without the `v` prefix.

## Update behaviour

ProtectedCore checks the manifest after the configured initial delay and then at the configured interval. A newer DLL is downloaded, SHA-256 verified, backed up, and staged. The running extension is not hot-swapped. Stop the server, run the staged `Apply-RogueRustUpdate.ps1`, and restart the server.
