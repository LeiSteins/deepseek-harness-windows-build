# DeepSeek Harness Windows 客户端封装

本项目基于 [DeepSeek Harness 官方客户端源码](https://github.com/deepseek-ai/deepseek-harness)，将其封装为可直接安装使用的 **Windows x64 桌面客户端**。

**下载 `.exe` 安装包即可安装，无需额外安装 Node.js、pnpm 等运行或构建依赖，也无需自行下载源码、配置开发环境。** 安装包包含客户端运行所需的环境，构建过程由 GitHub Actions 自动完成。

本仓库为独立维护的 Windows 打包项目，使用官方客户端源码构建，非 DeepSeek 官方发布渠道。

## 下载与安装

1. 打开 [最新构建下载页面](https://github.com/LeiSteins/deepseek-harness-windows-build/releases/tag/windows-feed)。
2. 在页面的 **Assets** 中下载 `deepseek-harness-<version>-win-x64-unsigned.exe` 安装包。
3. 运行安装包，按提示完成安装，然后启动 DeepSeek Harness。

上游项目处于早期开发阶段，版本会持续以 **Pre-release（预发布）** 状态发布。上述链接固定指向随构建更新的 `windows-feed` 页面，不依赖 Latest 正式版入口。如需指定版本或历史版本，请查看 [全部版本（含预发布）](https://github.com/LeiSteins/deepseek-harness-windows-build/releases)。

日常安装只需下载 `.exe` 文件，无需下载源码、`.blockmap` 或 `nightly.yml`。安装包不包含 API Key 或用户凭据，使用时按客户端提示完成所需配置。

安装包尚未进行代码签名，Windows 可能显示 Microsoft Defender SmartScreen 提示。

## 自动更新

客户端内置指向本仓库的更新源。检测到新版本后，可下载更新，并在确认后安装。

客户端会在启动时检查更新，之后大约每 10 分钟检查一次。侧边栏宽度足够时，更新状态会显示在账户按钮旁；没有新版本且检查正常时，不显示状态提示。

更新由本仓库的 `windows-feed` Release 提供，不使用 DeepSeek 官方更新服务。

### 旧版本升级

更新源启用前构建的版本（`0.1.6-alpha.2` 及更早版本）不包含 `app-update.yml`，无法自动更新。请从 [最新构建下载页面](https://github.com/LeiSteins/deepseek-harness-windows-build/releases/tag/windows-feed) 下载新版安装包并安装，后续即可使用内置更新功能。

## 版本同步与构建

GitHub Actions 每天北京时间（UTC+8）00:00 检查官方仓库最新的非草稿 Release。如果本仓库尚无对应版本的安装包，就自动构建并发布，沿用官方版本名称和标签。

维护者也可以在 Actions 页面选择 **Run workflow**，手动重新构建最新官方版本。手动构建会替换对应 Release 中已有的安装包。

每次成功构建还会保留 30 天的工作流产物，包括 NSIS 安装包、差分下载所需的 `.blockmap`、更新清单 `nightly.yml`，以及记录上游源码提交的 `BUILD-INFO.txt`。体积较大的中间目录 `win-unpacked` 不会上传。

## 构建与更新源维护

以下内容供维护构建流程时参考，普通用户直接下载安装包即可。

工作流使用官方提供的 Windows x64 无签名打包命令：

```text
pnpm run package:desktop:win:x64:unsigned
```

构建时根据上游公开示例生成 `apps/desktop/.env.windows`，仅替换应用 ID，签名及上传凭据字段保持为空。构建环境所需的 Node.js、pnpm 等依赖由工作流配置，不需要用户在本机安装。

### 更新源配置

安装包的 `resources/app-update.yml` 中包含以下更新源：

```yaml
provider: generic
url: https://github.com/LeiSteins/deepseek-harness-windows-build/releases/download/windows-feed/
channel: nightly
```

上游默认关闭 `--unsigned` 构建的更新功能，因此工作流会调整 `apps/desktop/scripts/electron-builder-config.mjs`，保留指向本仓库的 generic 发布配置。构建会校验是否生成 `nightly.yml`，以及打包后的应用是否包含匹配的 `app-update.yml`；缺少任一项都会导致构建失败。

每次构建会将以下三个文件发布到滚动更新的 `windows-feed` Release。该 Release 标记为预发布，避免成为仓库的 Latest 版本。

| 文件 | 用途 |
| --- | --- |
| `nightly.yml` | 客户端读取的更新清单 |
| `deepseek-harness-<version>-win-x64-unsigned.exe` | 更新所用的无签名安装包 |
| `deepseek-harness-<version>-win-x64-unsigned.exe.blockmap` | 支持差分下载 |

以上为当前版本的文件命名；早期版本可能不带 `-unsigned` 后缀，下载时以对应 Release 中的实际文件名为准。

发布时先上传新版本安装包和 `.blockmap`，再更新 `nightly.yml`。全部上传成功后，自动删除 `windows-feed` 中旧版本的 Windows x64 安装包及 `.blockmap`，仅保留当前构建的更新文件。按版本标签发布的历史 Release 不受影响。

更新器根据 `nightly.yml` 的相对路径查找安装包和 `.blockmap`，三个文件必须保留在同一个 Release 中。**不要重命名或删除 `windows-feed` 标签**，因为已安装客户端的更新地址已固定。若需迁移更新源，应修改工作流中的 `DSH_WINDOWS_FEED_TAG`，并发布保留旧地址可用的过渡版本。

更新检查周期可通过 `DSH_DESKTOP_UPDATE_CHECK_INTERVAL_MS`、`DSH_DESKTOP_UPDATE_CHECK_MAX_BACKOFF_MS` 和 `DSH_DESKTOP_UPDATE_CHECK_JITTER` 调整。

更多打包细节请参阅上游的 [桌面客户端文档](https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/desktop/README.md)。
