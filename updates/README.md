# PieOS update channel

`updates/channel.json` is the manifest polled by the PieOS auto-updater (`/usr/local/sbin/pieos-update`, from PieOS 1.1 / weekly item 010):

```
https://raw.githubusercontent.com/mettir2019-lgtm/pieos/main/updates/channel.json
```

## Format (schema 1)

| Field | Meaning |
|---|---|
| `latest` | newest version on this channel |
| `releases[]` | one entry per GitHub Release (`vX.Y`), newest last |
| `releases[].version`, `date`, `tag`, `release_url`, `notes` | release info |
| `releases[].iso` | full ISO: `name`, `size`, `sha256`, `sha256_url`, and `parts[]` (`name`, `url`, `size`, `sha256`) for the split download |
| `releases[].pack` | update pack for installed PCs, or `null`: `name`, `url`, `size`, `sha256` |

An update pack is a `.tar.gz` with one top-level folder containing `VERSION` and `features/NNN-name/install.sh` (+ `files/`). Packs are **cumulative**: each one contains every feature so far. Each PC runs every feature id **once**, in order, and records it in `/var/lib/pieos/features/applied`.

The updater downloads the newest release's `pack` from the GitHub Release assets, checks its **size and SHA256** against this manifest and refuses it on any mismatch, then applies new features. It never re-installs the ISO; new ISOs are only for fresh installs.

Every pack is also uploaded as `pieos-features.tar.gz`, so PCs still on the 1.0 updater can use
`FEATURES_URL=https://github.com/mettir2019-lgtm/pieos/releases/latest/download/pieos-features.tar.gz`.
That legacy path has no SHA256 check. The first pack includes item 010, which moves the PC to this manifest.

**Security:** feature scripts run as root. The SHA256 protects against corrupt or tampered *downloads*, but the manifest and the packs come from the same GitHub account. Keep 2FA on the account and review every pack before you publish it.
