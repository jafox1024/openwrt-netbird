# openwrt-netbird

[![GitHub Actions](https://github.com/jafox1024/openwrt-netbird/workflows/build/badge.svg?branch=main)](https://github.com/jafox1024/openwrt-netbird/actions?query=workflow%3Abuild)

OpenWrt package for [netbird](https://github.com/netbirdio/netbird)

## 分支布局

仓库默认分支为 `main`，对应 OpenWrt 25.12 主线；24.10 线保留为备份分支。

| 分支 | OpenWrt | 包管理器 / 包格式 | 架构 | 状态 |
| --- | --- | --- | --- | --- |
| **`main`** | 25.12.x（SDK 25.12.5） | apk / `.apk` | x86_64, rockchip_armv8 | 主线，跟随上游自动更新 |
| `openwrt-24.10` | 24.10.x（SDK 24.10.8） | opkg / `.ipk` | x86_64, rockchip_armv8 | 备份线，内容冻结 |

> OpenWrt 25.12 起包管理器由 **opkg 换成 apk**，`.ipk` 已停止使用，因此 25.12 设备必须使用 `main` 产出的 `.apk`。

## 安装

下载 Release 中对应架构的 `.apk`（文件名带架构后缀，如 `netbird-0.78.1-r1-x86_64.apk`），拷到设备上安装：

```bash
apk add --allow-untrusted /tmp/netbird-0.78.1-r1-x86_64.apk
```

注意 `--allow-untrusted`：本包未加入 OpenWrt 官方签名链，属于本地/第三方包。

OpenWrt 24.10（opkg）请改用 `openwrt-24.10` 分支发布的 `.ipk`：

```bash
opkg install /tmp/netbird_0.78.1-r1_x86_64.ipk
```

## 使用

用 setup key 登录 peer：

```bash
netbird login --setup-key <SETUP_KEY>
```

启动 netbird 后台服务：

```bash
/etc/init.d/netbird enable
/etc/init.d/netbird start
```

## 构建与发布（GitHub Actions）

流水线定义在 `.github/workflows/build.yml`：

1. `check` job：拉取 `netbirdio/netbird` 最新 release tag，比对 `netbird/Makefile` 的 `PKG_VERSION`；有更新就改版本号与 `PKG_HASH`、提交并推回 `main`。
2. `build` job：矩阵编译 `x86_64`、`rockchip_armv8` 两个架构（OpenWrt SDK 25.12.5）。编译前用 [sbwml/packages_lang_golang](https://github.com/sbwml/packages_lang_golang) 的 `26.x`（Go 1.26）feed 替换官方 `lang/golang`——netbird 0.78.1 的 `go.mod` 要求 `go >= 1.26`。
3. 产物同时上传到 Actions Artifacts 和 Release（tag 形如 `v0.78.1-openwrt25.12`）。

触发方式：

- **自动**：`main` 是默认分支，每天 00:00 UTC 检查一次上游版本，有新版本才构建并发布。
- **手动**：`Actions` → `build` → `Run workflow` → 选择分支。手动触发会直接构建并发布，不要求版本有变化。

### 备份线（OpenWrt 24.10）

`openwrt-24.10` 分支冻结在 24.10.8 + opkg/ipk 的状态，不参与每日自动更新。需要重新出 24.10 的包时，
在 `Actions` → `build` → `Run workflow` 里选择 `openwrt-24.10` 分支运行，产物 tag 为 `v<netbird版本>`。

### 升级 OpenWrt SDK

改 `.github/workflows/build.yml` 中这几处即可：

- `env.OPENWRT_RELEASE` / `env.OPENWRT_SERIES`（用于产物名与 Release tag 后缀）
- matrix 里两个 `sdk` 下载地址（文件名含 gcc 版本，从
  `https://downloads.openwrt.org/releases/<版本>/targets/<target>/` 目录页取准确文件名）
