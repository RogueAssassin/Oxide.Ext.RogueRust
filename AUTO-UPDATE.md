# RogueRust automatic updates

RogueRust can update itself from this repository's releases. It is on by default.

## How it works

1. When the extension starts, and then every `checkIntervalMinutes`, RogueRust reads `RogueRust.manifest.json` from the latest release:
   `https://github.com/RogueAssassin/Oxide.Ext.RogueRust/releases/latest/download/RogueRust.manifest.json`
2. If the manifest's version is newer than the installed one, it checks the manifest's RSA-SHA256 signature against the public key built into the DLL. If the signature doesn't verify, it doesn't install the update, and it logs an error.
3. It downloads `Oxide.Ext.RogueRust.dll` from the same release and checks that its size and SHA-256 match the manifest.
4. It renames the installed DLL to `Oxide.Ext.RogueRust.dll.backup` and writes the new one in its place. If any of that fails, the backup is moved back.
5. It logs `[RogueRust] Version X installed. Restart the server to load it.`

The running server keeps using the version it started with. The update takes effect on the **next restart**.

Pre-release and testing builds are never offered. GitHub's "latest release" is always the newest stable release.

## Settings

Settings are in `RogueRust.Update.json` in the Oxide or Carbon config folder (`oxide/config/` or `carbon/configs/`). The file is created with these defaults the first time the extension loads:

```json
{
  "enabled": true,
  "automaticDownload": true,
  "checkIntervalMinutes": 60,
  "repository": "RogueAssassin/Oxide.Ext.RogueRust",
  "manifestAsset": "RogueRust.manifest.json",
  "requireSignedManifest": false,
  "manifestPublicKeyXml": ""
}
```

| Setting | Effect |
|---|---|
| `enabled` | `false` turns update checks off completely. |
| `automaticDownload` | `false` only logs that a new version is available; nothing is downloaded. |
| `checkIntervalMinutes` | How often to check after startup. The minimum is 15. |
| `repository`, `manifestAsset` | Where updates come from. Leave these as they are. |
| `requireSignedManifest`, `manifestPublicKeyXml` | Release builds always require a signed manifest, checked against the key built into the DLL, so these don't need changing. |

Changes to the file take effect the next time the extension loads.

## Checking manually

Run this in the server console:

```text
roguerust.update
```

It needs server administrator access (auth level 2) when run by a player; the server console and RCON always have it. The result appears in the server log.

## What a release contains

| File | Purpose |
|---|---|
| `Oxide.Ext.RogueRust.dll` | The extension |
| `RogueRust.manifest.json` | Version, download URL, size, SHA-256 and signature that the updater reads |
| `SHA256SUMS.txt` | Checksum for checking a manual download |

To check a manual download on Windows:

```powershell
Get-FileHash .\Oxide.Ext.RogueRust.dll -Algorithm SHA256
```

The hash should match the line in `SHA256SUMS.txt`.

## Rolling back

Stop the server, delete `Oxide.Ext.RogueRust.dll` and rename `Oxide.Ext.RogueRust.dll.backup` back to `Oxide.Ext.RogueRust.dll`. Set `"enabled": false` in `RogueRust.Update.json` first if you don't want it updated again straight away. Then start the server.
