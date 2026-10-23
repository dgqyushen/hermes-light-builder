# Hermes Light Builder

Unofficial CI that builds the **`light` (remote-only) variant** of
[Hermes Desktop](https://hermes-agent.nousresearch.com/desktop) for Windows x64,
and publishes it as a GitHub Release.

上游把 `light` 作为构建目标保留，但没有发布计划。本仓库只做一件事：
按上游 tag 编译出 Windows x64 的 light 客户端，发到 Releases。

---

## 中文

### 这是什么

Hermes Desktop 官方的安装包会先把完整 agent（Python、依赖树、CLI）装到本机，
体积约 4.6 GB。`light` 变体只有 Electron 外壳加聊天界面，不带 Python、不带本地
agent、不带 payload，解包约 400 MB，压缩后约 190 MB，通过 Settings → Gateways →
Remote gateway 连到你自己的后端。

上游代码里这个变体是完整的（`HERMES_DESKTOP_VARIANT=light`），但官方明确表示不会
单独发布（[#102564](https://github.com/NousResearch/hermes-agent/pull/102564)、
[#38519](https://github.com/NousResearch/hermes-agent/issues/38519)）。本仓库把
这件事自动化：每天检查上游最新 stable tag，没有对应产物就编译一份。

### 怎么用

1. 打开 [Releases](../../releases)，下载最新的
   `Hermes-Light-<tag>-win-x64.zip` 和 `.sha256`。
2. 校验：`certutil -hashfile <zip> SHA256`。
3. **完整解压**（不要直接在压缩包里运行），双击 `Hermes Light.exe`。
4. 首次启动进 Settings → Gateways → Remote gateway，填你的后端地址
   （例如 `http://<tailscale-ip>:9119`），登录后保存。

### 已知限制

- **未签名。** 没有 Authenticode 证书。SmartScreen 会弹警告（更多信息 → 仍要运行）；
  开了 Smart App Control 的机器会直接拦截，自签名也过不了 SAC。
- **没有自动更新。** light 变体的更新通道上游未发布，`updateMechanism` 是
  `external`。换版本就是重新下载下一个 release。
- **只有 Windows x64。** macOS 的 dmg 必须在 macOS 上打，Linux 的 AppImage 必须在
  Linux 上打。本仓库不包含这两条 leg。
- **非官方。** 产物未经过 Nous Research 审核。源码是上游原样，构建过程在
  Actions 日志里公开可查。

### 怎么触发构建

- 自动：每天 03:17（北京时间）检查上游最新 stable tag，已有 release 就跳过。
- 手动：Actions → Build Hermes Light → Run workflow。
  - `version` 留空 = 最新 stable；填 `v0.22.5` = 指定 tag。
  - `force` 勾上 = 即使已有同名 release 也重新编译。

### 构建过程

```
windows-latest (4C/16G/14G SSD, 一次性虚拟机)
  └─ git clone --depth 1 --branch <tag> upstream
  └─ npm ci --workspace apps/desktop --workspace apps/shared
  └─ HERMES_DESKTOP_VARIANT=light npm run pack --workspace apps/desktop
  └─ 断言 resources/agent-payload 不存在
  └─ 7z 打包 + SHA256
  └─ gh release create/upload
```

峰值磁盘约 2.5 GB，跑完随虚拟机销毁，不落在任何人的机器上。

---

## English

### What this is

The official Hermes Desktop installer bootstraps the full agent payload on the
client machine (~4.6 GB). The `light` variant is the Electron shell plus chat
surface only — no Python, no local agent, no payload. Roughly 400 MB unpacked,
190 MB zipped. It connects to a backend you run elsewhere via
**Settings → Gateways → Remote gateway**.

Upstream keeps `light` as a working build target (`HERMES_DESKTOP_VARIANT=light`)
but has said it will not publish one
([#102564](https://github.com/NousResearch/hermes-agent/pull/102564),
[#38519](https://github.com/NousResearch/hermes-agent/issues/38519)). This repo
automates the gap: every day it checks the latest stable upstream tag and builds
a Windows x64 client if none exists yet.

### Usage

1. Grab the newest `Hermes-Light-<tag>-win-x64.zip` and `.sha256` from
   [Releases](../../releases).
2. Verify: `certutil -hashfile <zip> SHA256`.
3. **Extract the whole archive** (do not run from inside the ZIP), then launch
   `Hermes Light.exe`.
4. On first run: Settings → Gateways → Remote gateway, enter your backend URL
   (e.g. `http://<tailscale-ip>:9119`), sign in, save.

### Known limitations

- **Unsigned.** No Authenticode certificate. SmartScreen warns; Smart App Control
  blocks it outright, and a self-signed certificate does not satisfy SAC either.
- **No auto-update.** Upstream publishes no update feed for `light`
  (`updateMechanism` is `external`). Upgrading means downloading the next release.
- **Windows x64 only.** macOS dmg must be built on macOS; Linux AppImage must be
  built on Linux. This repo has no leg for either.
- **Unofficial.** Not reviewed by Nous Research. Source is upstream unmodified;
  the build is fully visible in the Actions log.

### Triggering a build

- Automatic: daily at 19:17 UTC (03:17 Asia/Shanghai), skipped when a release for
  that tag already exists.
- Manual: Actions → Build Hermes Light → Run workflow.
  - `version` blank = latest stable; `v0.22.5` = pin a tag.
  - `force` = rebuild even if the release already exists.

### What the job does

```
windows-latest (4C/16G/14G SSD, disposable VM)
  └─ git clone --depth 1 --branch <tag> upstream
  └─ npm ci --workspace apps/desktop --workspace apps/shared
  └─ HERMES_DESKTOP_VARIANT=light npm run pack --workspace apps/desktop
  └─ assert resources/agent-payload is absent
  └─ 7z archive + SHA256
  └─ gh release create/upload
```

Peak disk is about 2.5 GB and dies with the VM.

### Why GitHub Actions

Standard runners are free and unmetered for public repositories, including
Windows and macOS. The build produces several GB of `node_modules` and caches;
on a hosted runner that litter is discarded with the VM instead of accumulating
on your own disk.

---

## License

The build recipe here is MIT. The resulting binaries are builds of
[Hermes Agent](https://github.com/NousResearch/hermes-agent), also MIT,
copyright Nous Research.
