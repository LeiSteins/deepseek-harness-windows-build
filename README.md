# DeepSeek Harness Windows Builds

This repository builds the Windows x64 desktop application from the official
[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) source code
with GitHub Actions.

## Download a build

Open the repository's **Releases** page and download the `.exe` asset from the
release matching the official DeepSeek Harness release name and tag.

Each successful build also keeps a workflow artifact for 30 days. It contains
only the NSIS installer, its blockmap, and `BUILD-INFO.txt` with the exact
upstream commit used for the build. The large intermediate `win-unpacked`
directory is not uploaded.

## Automatic and manual builds

The workflow runs every day at 00:00 Asia/Shanghai time. It reads the latest
non-draft release from the official DeepSeek Harness repository and builds it
only when this repository does not already contain a release with the same
tag.

Choose **Run workflow** in the Actions tab to force a new build of the latest
official DeepSeek Harness release. Manual builds use the same upstream release
name and tag, and replace an existing installer asset with the newly built
file.

## Automatic updates

Each installer built here carries an update source in
`resources/app-update.yml`, so an installed application can detect newer
builds, download them, and install them after a confirmation:

```yaml
provider: generic
url: https://github.com/LeiSteins/deepseek-harness-windows-build/releases/download/windows-feed/
channel: nightly
```

Upstream disables its packaged updater for `--unsigned` builds, so the workflow
patches `apps/desktop/scripts/electron-builder-config.mjs` to keep a generic
publish target pointed at this repository instead. The build then fails loudly
unless electron-builder produced `nightly.yml` **and** the packaged application
contains a matching `app-update.yml`.

Every build publishes three assets to the rolling `windows-feed` release
(marked as a prerelease so it never becomes the repository's "Latest"):

| Asset | Purpose |
| --- | --- |
| `nightly.yml` | Feed read by installed applications |
| `deepseek-harness-<version>-win-x64.exe` | Installer offered to updating clients |
| `deepseek-harness-<version>-win-x64.exe.blockmap` | Differential download support |

The updater resolves the installer and blockmap relative to `nightly.yml`, so
all three assets must stay in that one release. The URL is baked into every
installer, which means **the feed tag must not be renamed or deleted** — doing
so strands existing installations. To move the feed elsewhere, change
`DSH_WINDOWS_FEED_TAG` in the workflow and publish one transitional release that
keeps the old URL alive.

Applications check at startup and then roughly every 10 minutes; the update
state appears beside the account button in the sidebar while the sidebar is
wide enough to show it. `DSH_DESKTOP_UPDATE_CHECK_INTERVAL_MS`,
`DSH_DESKTOP_UPDATE_CHECK_MAX_BACKOFF_MS`, and `DSH_DESKTOP_UPDATE_CHECK_JITTER`
tune that schedule. An idle check renders nothing at all, so a working feed with
no newer version looks the same as an application that never checks — only a
failure or an available version is visible.

### Updating an older installation

Builds made before the feed existed (0.1.6-alpha.2 and earlier) contain no
`app-update.yml` and therefore cannot update themselves. Either install a newer
build once, or add the update source to the existing installation and restart
the application:

```powershell
$resources = Join-Path $env:LOCALAPPDATA 'Programs\DeepSeek Harness\resources'
@'
provider: generic
url: https://github.com/LeiSteins/deepseek-harness-windows-build/releases/download/windows-feed/
channel: nightly
updaterCacheDirName: "@deepseek-aidsh-desktop-updater"
'@ | Set-Content -Path (Join-Path $resources 'app-update.yml') -Encoding utf8NoBOM
```

The feed only advertises versions above the installed one, so this is safe to
add to any installation that already has a feed-less build.

## Important notice

The generated installer is unsigned. Windows may show a Microsoft Defender
SmartScreen warning. This repository is an independent build helper and is not
an official DeepSeek distribution. No API key or user credential is embedded
in the installer. The built-in updater points at this repository's own
`windows-feed` release, never at a DeepSeek update service.

The workflow runs DeepSeek Harness's official unsigned packaging command:

```text
pnpm run package:desktop:win:x64:unsigned
```

For current upstream versions, the workflow derives the required
`apps/desktop/.env.windows` file from the public upstream example and replaces
only the application ID. Signing and upload credential fields remain empty.

See the upstream
[Desktop documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.md)
for packaging details and requirements.
