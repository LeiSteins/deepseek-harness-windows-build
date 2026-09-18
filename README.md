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

## Important notice

The generated installer is unsigned. Windows may show a Microsoft Defender
SmartScreen warning. This repository is an independent build helper and is not
an official DeepSeek distribution. No API key or user credential is embedded
in the installer.

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
