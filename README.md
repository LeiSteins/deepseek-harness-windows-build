# DeepSeek Harness Windows Builds

This repository builds the Windows x64 desktop application from the official
[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) source code
with GitHub Actions.

## Download a build

1. Open the repository's **Actions** tab.
2. Select **Build DeepSeek Harness for Windows**.
3. Open a successful workflow run.
4. Download the `deepseek-harness-windows-x64-*` artifact at the bottom of the
   run page.
5. Extract the artifact and run the included installer.

Artifacts are retained for 30 days. Each artifact contains only the NSIS
installer, its blockmap, and `BUILD-INFO.txt` with the exact upstream commit
used for the build. The large intermediate `win-unpacked` directory is not
uploaded.

## Build a branch, tag, or commit

Choose **Run workflow** in the Actions tab and enter one of the following in
the `ref` field:

- `master` for the latest development source;
- a release tag such as `dsh-v0.1.6-alpha.2`;
- a full upstream commit SHA for a reproducible build.

The workflow also builds the latest upstream `master` branch every Monday.

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
